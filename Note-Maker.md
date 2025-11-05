<%* await tp.file.include("[[./data/config]]");
'use strict';
// <!--- Note Auswahl, json Parsen und grundlegende Datentypen anlegen --->
const defaultJsonPath = '' + TemplatePath + '/data/default.json';
const fs = require('fs');
const defaultJson = JSON.parse( fs.readFileSync(defaultJsonPath, 'utf8'));
const noteMainRangePool = defaultJson.noteMainRangePool
let noteMainRange = await tp.system.suggester(noteMainRangePool.label, noteMainRangePool.value, false, noteMainRangePool.prompt);
const noteMainRangeSelected = noteMainRangePool[noteMainRange];
const noteMainRangeJson = JSON.parse ( fs.readFileSync(TemplatePath + noteMainRangeSelected.noteMainRangeValue, 'utf8'));
const noteTagsMap = defaultJson.noteTagsMap;
const noteBasicMap = defaultJson.noteBasicMap;
const noteSubTitleMap = defaultJson.noteSubTitleMap;
const docIconMap = noteMainRangeJson.docIconMap;
let key = 0;
// <!--- Note Datum eruiren --->
const noteDateMap = defaultJson.noteDateMap;
const noteDateNow = tp.date.now(noteBasicMap.dateFormat);

// <!--- Notiz Gültigkeit festlegen --->
const durabilityMap = defaultJson.durabilityMap;
let durabilityLabel = [durabilityMap.notePermanentlyLabel, durabilityMap.noteTemporaryLabel]
let durabilityValue = [durabilityMap.notePermanentlyValue, durabilityMap.noteTemporaryValue]
if (durabilityMap.active) {
  label = await tp.system.suggester(durabilityLabel, durabilityValue, false, durabilityMap.prompt);
};
durabilityValue = label;

// <!--- Wenn Gültigkeit befristet, wird ein Enddatum festlegen --->
const durabilityDateMap = defaultJson.durabilityDateMap;
const durabilitySignMap = defaultJson.durabilitySignMap
if (durabilityValue === durabilityMap.noteTemporaryValue) {
	durabilityDate = await tp.system.prompt(durabilityDateMap.prompt, tp.date.now(noteBasicMap.dateFormat, "P3M"));
	while (!durabilityDate){
    durabilityDate = await tp.system.prompt(durabilityDateMap.prompt, tp.date.now(noteBasicMap.dateFormat, "P3M"));
    };
	noteSign = durabilitySignMap.noteSignTemporary;
} else {
  noteSign = durabilitySignMap.noteSignPermanently
};

// <!--- Notiz Name festlegen und erstellen --->
const docTitleMap = defaultJson.docTitleMap
let title = tp.file.title;
while (title.startsWith("Untitled") || !title){title = await tp.system.prompt(docTitleMap.titleFieldInsert);};

if (docIconMap.active && durabilitySignMap.active){
  await tp.file.rename(docIconMap.mark  + noteBasicMap.space + noteSign  + noteBasicMap.space + title);
} else if (docIconMap.active && !durabilitySignMap.active) {
  await tp.file.rename(docIconMap.mark + noteBasicMap.space + title);
} else if (!docIconMap.active && durabilitySignMap.active) {
  await tp.file.rename(noteSign + noteBasicMap.space + title);
} else {
  await tp.file.rename(title);
};

// <!--- Notiz Titel festlegen --->
const noteTitleMap = defaultJson.noteTitleMap;
let docTitle;
if (noteTitleMap.active) {
  while (!docTitle){docTitle = await tp.system.prompt(noteTitleMap.docTitleFieldInsert);};
};

// <!--- Notiz-Aliases festlegen --->
const noteAliasMap = defaultJson.noteAliasMap;
let aliasTitle = [];
if (noteAliasMap.active) {
  while (!aliasTitle[key]){aliasTitle[key] = await tp.system.prompt(noteAliasMap.aliasTitleFieldInsert0);};
  while (aliasTitle[key]){key++; aliasTitle[key] = await tp.system.prompt(noteAliasMap.aliasTitleFieldInsert1)};
};

// <!--- Notiz MetaPool eintragen --->
const noteMetaPool = defaultJson.noteMetaPool;
let noteMeta = [], noteMetaLabel = [], noteMetaKey = 0;
if (noteMetaPool.active) {
  for (const key in noteMetaPool) {
    if (Object.prototype.hasOwnProperty.call(noteMetaPool, key) && key.startsWith('noteMeta')) {
      noteMeta[noteMetaKey] = noteMetaPool[key]
      if (noteMetaPool[key].active) {
        label = await tp.system.suggester(noteMetaPool[key].label , noteMetaPool[key].value, false, noteMetaPool[key].prompt);
          while (label === noteMetaPool[key].value[noteMetaPool[key].manualEntryNumber] || !label){
            label = await tp.system.prompt(noteMetaPool[key].input);
            };
        noteMetaLabel[noteMetaKey] = (label);
        noteMetaKey++;
      };
    };
  };
};

// <!--- Dokument Kategorie und Unterkategorie erstellen ---
const noteDocMetaPool = noteMainRangeJson.noteDocMetaPool
let noteDocMeta = [], noteDocMetaLabel = [], noteDocMetaNonExist = [], noteDocMetaKey = 0;
if (noteDocMetaPool.active) {
  for (const key in noteDocMetaPool) {
    if (Object.prototype.hasOwnProperty.call(noteDocMetaPool, key) && key.startsWith('noteDocMeta')) {
      noteDocMeta[noteDocMetaKey] = noteDocMetaPool[key];
      if (noteDocMetaPool[key].active) {
        label = await tp.system.suggester(noteDocMetaPool[key].label , noteDocMetaPool[key].value, false, noteDocMetaPool[key].prompt);
        if (label === noteDocMetaPool[key].value[noteDocMetaPool[key].manualEntryNumber]) {
          label = await tp.system.prompt(noteDocMetaPool[key].input);
          while (!label){label = await tp.system.prompt(noteDocMetaPool[key].input);};
        };
        noteDocMetaLabel[noteDocMetaKey] = (label);
        noteDocMetaNonExist[noteDocMetaKey] = (noteDocMetaPool[key].value[noteDocMetaPool[key].nonExistEntryNumber]);
        noteDocMetaKey++;
      };
      let selected = noteMainRangeJson.noteDocMetaPool[key].second[label];
      let manualEntry = noteDocMetaPool[key].value[noteDocMetaPool[key].manualEntryNumber]
      if (selected && noteDocMetaPool[key].second.active && selected !== manualEntry) {
        label = await tp.system.suggester(selected.label, selected.value, false, selected.prompt);
        noteDocMetaLabel[noteDocMetaKey] = (label);
      };
    };
  };
};

// <!--- Erweiterte Notiz Informationen erstellen --->
const noteExpandPool = noteMainRangeJson.noteExpandPool;
let noteExpand = [], noteExpandLabel = [], noteExpandUnknown = [], noteExpandNonExist = [], noteExpandKey = 0;
if (noteExpandPool.active) {
  for (const key in noteExpandPool) {
    if (Object.prototype.hasOwnProperty.call(noteExpandPool, key) && key.startsWith('noteExpand')) {
      noteExpand[noteExpandKey] = noteExpandPool[key];
      if (noteExpandPool[key].active) {
        label = await tp.system.suggester(noteExpandPool[key].label , noteExpandPool[key].value, false, noteExpandPool[key].prompt);
          while (label === noteExpandPool[key].value[noteExpandPool[key].manualEntryNumber] || !label){
            label = await tp.system.prompt(noteExpandPool[key].input);
          };
        noteExpandLabel[noteExpandKey] = (label);
        noteExpandUnknown[noteExpandKey] = noteExpandPool[key].value[noteExpandPool[key].unknownEntryNumber];
        noteExpandNonExist[noteExpandKey] = noteExpandPool[key].value[noteExpandPool[key].nonExistEntryNumber];
        noteExpandKey++;
      };
    };
  };
};

// <!--- Externe Quellen und Nachweise erstellen --->
function validUrl(string) {
  try {
    new URL(string);
    return true;
  } catch (err) {
    return false;
    };
};
let extLink = noteBasicMap.extLink, urlHttp = [], sourcesTitle = [], sourcesUrl = [], sourceKey = 0, noteSourcesMap = defaultJson.noteSourcesMap;
if (noteSourcesMap && noteSourcesMap.active){
  while (!sourcesTitle[sourceKey]){
    sourcesTitle[sourceKey] = await tp.system.prompt(noteSourcesMap.sourcesTitleFieldInsert0);
    if (sourcesTitle[sourceKey]){
      while (!sourcesUrl[sourceKey]){
        sourcesUrl[sourceKey] = await tp.system.prompt(noteSourcesMap.sourcesUrlFieldInsert0);
      };
      if (!validUrl(sourcesUrl[sourceKey])){urlHttp[sourceKey] = "https://"}
    };
  };
  while (sourcesTitle[sourceKey]){
    sourceKey++;
    sourcesTitle[sourceKey] = await tp.system.prompt(noteSourcesMap.sourcesTitleFieldInsert1);
    if (sourcesTitle[sourceKey]){
      while (!sourcesUrl[sourceKey]){
        sourcesUrl[sourceKey] = await tp.system.prompt(noteSourcesMap.sourcesUrlFieldInsert0);
      };
      if (!validUrl(sourcesUrl[sourceKey])){urlHttp[sourceKey] = "https://"}
    };
  };
};

// <!--- Notiz-Teaser erstellen --->
const shortDescriptionMap = defaultJson.shortDescriptionMap;
let shortDescriptionContent;
while (shortDescriptionMap.active && !shortDescriptionContent) {
  shortDescriptionContent = await tp.system.prompt(shortDescriptionMap.prompt);
  };

// <!--- Notiz Ausgaben schreiben--->
tR += `---\n`;
// <!--- Erstellungsdatum der Notiz schreiben--->
if (noteDateMap.active) {
  tR += `${noteDateMap.createdOn} ${noteDateNow}\n`
};
if (durabilityMap.active) {tR += `${durabilityMap.printValue}: ${durabilityValue}\n`;};
if (durabilityMap.active && durabilityMap.active && durabilityValue === durabilityMap.noteTemporaryValue){
  tR += `${durabilityDateMap.printValue}: ${durabilityDate}\n`;
};

// <!--- Notiz Optionen schreiben --->
if (noteMetaPool.active){
  for (let i = 0; i < noteMeta.length; i++) {tR += `${noteMeta[i].printValue}: ${noteMetaLabel[i]}\n`};
};
// <!--- Notiz-Aliases schreiben --->
if (noteAliasMap.active){
  tR += `${defaultJson.noteAliasMap.alias}:\n`
  for (let i = 0; i < aliasTitle.length; i++) {if (aliasTitle[i] !== ""){tR += `  - ${aliasTitle[i]}\n`};};
};
// <!--- Notiz-Tags schreiben --->
if (noteTagsMap.active) {
  tR += `${noteTagsMap.tags}:
  - ${noteMainRange}\n`;
};
// <!------------------------------------ Beginn Spielwiese ------------------------------------>
if (noteTagsMap.active && noteDocMetaPool.active) {
  for (let i = 0; i < noteDocMetaLabel.length; i++) {
    if (noteDocMetaLabel[i] !== noteDocMetaNonExist[i]){tR += `  - ${noteDocMetaLabel[i]}\n`};
  };
};
// <!------------------------------------- Ende Spielwiese ------------------------------------->
tR += `---\n`
if (noteTitleMap.active && docTitleMap.active || noteTitleMap.active && !docTitleMap.active) {
  tR += `# ${docTitle}\n---\n`
} else if (!noteTitleMap.active && docTitleMap.active) {
  tR += `# ${title}\n---\n`
};

// <!--- Untertitel und Notiz-Teaser schreiben --->
if (noteSubTitleMap.active && shortDescriptionMap.active) {
  tR += `\n## ${noteSubTitleMap.shortDescriptionTitle}\n---\n${shortDescriptionContent}\n`;
} else if (!noteSubTitleMap.active && shortDescription.active) {
  tR += `\n${shortDescriptionContent}\n`;
} else if (noteSubTitleMap.active && !shortDescriptionMap.active) {
  tR += `\n## ${noteSubTitleMap.shortDescriptionTitle}\n---\n`;
};

// <!--- Untertitel und erweiterte Informationen schreiben --->
if (noteSubTitleMap.active && noteExpandPool.active){
    // <!--- Untertitel und Tabellen Head schreiben --->
  tR += `\n## ${noteSubTitleMap.advancedInfoTitle}\n---\n<table class=tabelle1>
  <tr><th >Kategorie</th><th>Wert</th></tr>\n`;
  // <!--- Notiz Optionen in die Tabelle schreiben --->
  for (let i = 0; i < noteExpand.length; i++) {if (noteExpandLabel[i] !== noteExpandNonExist[i]){
    tR += `<tr><td>${noteExpand[i].printValue}:</td><td>${noteExpandLabel[i]}</td></tr>\n`;};
  };
  tR += `</table>\n`;
} else if (!noteSubTitleMap.active && noteExpandPool.active){
  tR += `\n<table class=tabelle1>
  <tr><th >Kategorie</th><th>Wert</th></tr>\n`;
  // <!--- Notiz Optionen in die Tabelle schreiben--->
  for (let i = 0; i < noteExpand.length; i++) {if (noteExpandLabel[i] !== noteExpandNonExist[i]){
    tR += `<tr><td>${noteExpand[i].printValue}:</td><td>${noteExpandLabel[i]}</td></tr>\n`;};
  };
  tR += `</table>\n`;
} else if (noteSubTitleMap.active && !noteExpandPool.active){
  tR += `\n## ${noteSubTitleMap.advancedInfoTitle}\n---\n`;
};

// <!--- Untertitel und externen Quellen und Nachweise schreiben --->
  if (noteSubTitleMap.active && noteSourcesMap.active){
  tR += `\n## ${noteSubTitleMap.sourceTitle}\n---\n`
    // <!--- Externen Quellen und Nachweise Schreiben --->
    for (let i = 0; i < sourcesUrl.length && i < urlHttp.length && i < sourcesTitle.length; i++){
      if (sourcesUrl[i] && !urlHttp[i]){tR += `${extLink}[${sourcesTitle[i]}](${sourcesUrl[i]})&ensp;&ensp;&ensp;`
      } else {
        tR += `${extLink}[${sourcesTitle[i]}](${urlHttp[i]}${sourcesUrl[i]})&ensp;&ensp;&ensp;`
        };
    };
} else if (!noteSubTitleMap.active && noteSourcesMap.active){
  // <!--- Externen Quellen und Nachweise schreiben --->
  for (let i = 0; i < sourcesUrl.length && i < urlHttp.length && i < sourcesTitle.length; i++){
    if (sourcesUrl[i] && !urlHttp[i]){
      tR += `${extLink}[${sourcesTitle[i]}](${sourcesUrl[i]})&ensp;&ensp;&ensp;`
      } else {tR += `${extLink}[${sourcesTitle[i]}](${urlHttp[i]}${sourcesUrl[i]})&ensp;&ensp;&ensp;`
      };
  };
} else if (noteSubTitleMap.active && !noteSourcesMap.active){
  tR += `\n## ${noteSubTitleMap.sourceTitle}\n---\n`;
};

// <!--- Untertitel und interne Verweise schreiben --->
const intRefDefault = defaultJson.internalReferencesMap;
const intRefMainRange = noteMainRangeJson.internalReferencesMap;
if (noteSubTitleMap.active && intRefDefault.active || intRefMainRange.active) {
  // <!--- Untertitel schreiben --->
  tR += `\n\n## ${noteSubTitleMap.internalSourceTitle}\n---\n`;
  // <!--- Interne Verweise schreiben --->
  if (durabilityMap.active && intRefDefault.active && intRefDefault.durability.active) {
    tR += `[[${durabilityValue}]], `;
  if (noteMetaPool.active && intRefDefault.active && intRefDefault.noteMeta.active) {
    intRefDefault.noteMeta.select.map(i => noteMetaLabel[i]);
    intRefMainRange.noteExpand.select.forEach(i => { tR += `[[${noteMetaLabel[i]}]], ` });
  };
  if (intRefDefault.active) {
    tR += `[[${noteMainRange}]], `;
  };
// <!------------------------------------ Beginn Spielwiese ------------------------------------>
  if (noteDocMetaPool.active && intRefMainRange.active && intRefMainRange.noteDocMeta.active) {
    for (let i = 0; i < noteDocMetaLabel.length; i++) {if (noteDocMetaLabel[i] !== noteDocMetaNonExist[i]){tR += `[[${noteDocMetaLabel[i]}]], `};};
  };

// <!------------------------------------- Ende Spielwiese ------------------------------------->
  if (noteExpandPool.active && intRefMainRange.active && intRefMainRange.noteExpand.active) {
    for (let i = 0; i < noteExpand.length; i++) {
      if (noteExpandLabel[i] !== noteExpandUnknown[i] && noteExpandLabel[i] !== noteExpandNonExist[i]) {tR += `[[${noteExpandLabel[i]}]], `;}
    };
    // intRefMainRange.noteExpand.select.map(i => noteExpandLabel[i]);
    // intRefMainRange.noteExpand.select.forEach(i => {
      // if (noteExpandLabel[i] !== noteExpandUnknown[i] || noteExpandLabel[i] !== noteExpandNonExist[i]) {tR += `[[${noteExpandLabel[i]}]], `;}
    // });
  };
};
} else if (!noteSubTitleMap.active && intRefDefault.active || intRefMainRange.active) {
  // <!--- Interne Verweise schreiben --->
  if (durabilityMap.active && intRefDefault.active && intRefDefault.durability.active) {
    tR += `[[${durabilityValue}]], `;
  };
  if (noteMetaPool.active && intRefDefault.active && intRefDefault.noteMeta.active) {
    intRefDefault.noteMeta.select.map(i => noteMetaLabel[i]);
    intRefDefault.noteMeta.select.forEach(i => { tR += `[[${noteMetaLabel[i]}]], ` });
  };
  if (intRefDefault.active) {
    tR += `[[${noteMainRange}]], `;
  };
  // <!------------------------------------ Beginn Spielwiese ------------------------------------>
  if (noteDocMetaPool.active && intRefMainRange.active && intRefMainRange.noteDocMeta.active) {
    for (let i = 0; i < noteDocMetaLabel.length; i++) {if (noteDocMetaLabel[i] !== noteDocMetaNonExist[i]){tR += `[[${noteDocMetaLabel[i]}]], `};};
  };

// <!------------------------------------- Ende Spielwiese ------------------------------------->
  if (noteExpandPool.active && intRefMainRange.active && intRefMainRange.noteExpand.active) {
    for (let i = 0; i < noteExpand.length; i++) {
      if (noteExpandLabel[i] !== noteExpandUnknown[i] && noteExpandLabel[i] !== noteExpandNonExist[i]) {tR += `[[${noteExpandLabel[i]}]], `;}
    };
    // intRefMainRange.noteExpand.select.map(i => noteExpandLabel[i]);
    // intRefMainRange.noteExpand.select.forEach(i => {
    //   if (noteExpandLabel[i] !== noteExpandUnknown[i] || noteExpandLabel[i] !== noteExpandNonExist[i]) {tR += `[[${noteExpandLabel[i]}]], `;}
    // });
  };
} else if (noteSubTitleMap.active && !intRefDefault.active && !intRefMainRange.active) {
  tR += `\n\n## ${noteSubTitleMap.internalSourceTitle}\n---\n`;
};

// <!------------------------------------ Beginn Spielwiese ------------------------------------>
// <!------------------------------------- Ende Spielwiese ------------------------------------->
%>
