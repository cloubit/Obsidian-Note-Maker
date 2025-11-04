<%* await tp.file.include("[[./data/config]]");
"use strict";
// <!--- Datei default.json Parsen --->
const defaultJsonPath = '' + TemplatePath + '/data/default.json';
const fs = require('fs');
const defaultJson = JSON.parse( fs.readFileSync(defaultJsonPath, 'utf8'));

// <!--- Note Auswahl festlegen und selektierte JSON Datei parsen --->
let noteMainRangePool = defaultJson.noteMainRangePool
noteMainRange = await tp.system.suggester(noteMainRangePool.label, noteMainRangePool.value, false, noteMainRangePool.prompt);
const noteMainRangeSelected = noteMainRangePool[noteMainRange];
const noteMainRangeJson = JSON.parse ( fs.readFileSync(TemplatePath + noteMainRangeSelected.noteMainRangeValue, 'utf8'));

// <!--- variablen initialisieren --->
let key = 0, noteTagsMap = defaultJson.noteTagsMap, noteBasicMap = defaultJson.noteBasicMap, noteSubTitleMap = defaultJson.noteSubTitleMap, docIconMap = noteMainRangeJson.docIconMap;

// <!--- Note Datum eruiren --->
let noteDateMap = defaultJson.noteDateMap;
const noteDateNow = tp.date.now(noteBasicMap.dateFormat);

// <!--- Notiz Gültigkeit festlegen --->
let durabilityMap = defaultJson.durabilityMap;
durabilityLabel = [durabilityMap.notePermanentlyLabel, durabilityMap.noteTemporaryLabel]
durabilityValue = [durabilityMap.notePermanentlyValue, durabilityMap.noteTemporaryValue]
if (durabilityMap.active) {
  label = await tp.system.suggester(durabilityLabel, durabilityValue, false, durabilityMap.prompt);
};
durabilityValue = label;

// <!--- Wenn Gültigkeit befristet, wird ein Enddatum festlegen --->
let durabilityDateMap = defaultJson.durabilityDateMap;
let durabilitySignMap = defaultJson.durabilitySignMap
if (durabilityValue === durabilityMap.noteTemporaryValue) {
	durabilityDate = await tp.system.prompt(durabilityDateMap.prompt, tp.date.now(noteBasicMap.dateFormat, "P3M"));
	while (!durabilityDate){durabilityDate = await tp.system.prompt(durabilityDateMap.prompt, tp.date.now(noteBasicMap.dateFormat, "P3M"));};
	noteSign = durabilitySignMap.noteSignTemporary;
} else {
  noteSign = durabilitySignMap.noteSignPermanently
  };

// <!--- Notiz Name festlegen und erstellen --->
let docTitleMap = defaultJson.docTitleMap
title = tp.file.title;
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
let noteTitleMap = defaultJson.noteTitleMap, docTitle;
if (noteTitleMap.active) {
  while (!docTitle){docTitle = await tp.system.prompt(noteTitleMap.docTitleFieldInsert);};
};

// <!--- Notiz-Aliases festlegen --->
let aliasTitle = [], noteAliasMap = defaultJson.noteAliasMap;
if (noteAliasMap.active) {
  while (!aliasTitle[key]){aliasTitle[key] = await tp.system.prompt(noteAliasMap.aliasTitleFieldInsert0);};
  while (aliasTitle[key]){key++; aliasTitle[key] = await tp.system.prompt(noteAliasMap.aliasTitleFieldInsert1)};
};

// <!--- Notiz MetaPool eintragen --->
let noteMeta = [], noteMetaLabel = [], noteMetaKey = 0, noteMetaPool = defaultJson.noteMetaPool;
if (noteMetaPool.active) {
  for (const key in noteMetaPool) {
    if (Object.prototype.hasOwnProperty.call(noteMetaPool, key) && key.startsWith('noteMeta')) {
      noteMeta[noteMetaKey] = noteMetaPool[key]
      if (noteMetaPool[key].active) {
        label = await tp.system.suggester(noteMetaPool[key].label , noteMetaPool[key].value, false, noteMetaPool[key].prompt);
          while (label === noteMetaPool[key].value[noteMetaPool[key].manualEntryNumber] || !label){label = await tp.system.prompt(noteMetaPool[key].input);};
        noteMetaLabel[noteMetaKey] = (label);
        noteMetaKey++;
      };
    };
  };
};

// <!--- Dokument Kategorie und Unterkategorie erstellen ---
let noteDocMeta = [], noteDocMetaLabel = [], noteDocMetaLabel1 = [], noteDocMetaKey = 0, noteDocMetaPool = noteMainRangeJson.noteDocMetaPool;
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
        noteDocMetaKey++;
      };
      if (noteDocMetaPool[key].second.active) {
        const selected = noteMainRangeJson.noteDocMetaPool[key].second[label];
        if (selected) {label = await tp.system.suggester(selected.label, selected.value, false, selected.prompt);};
        noteDocMetaLabel[noteDocMetaKey] = (label);
        noteDocMetaLabel1[noteDocMetaKey]
        noteDocMetaKey++;
      };
    };
  };
};

// <!--- Erweiterte Notiz Informationen erstellen --->
let noteExpand = [], noteExpandLabel = [], noteExpandKey = 0, noteExpandPool = noteMainRangeJson.noteExpandPool;
if (noteExpandPool.active) {
  for (const key in noteExpandPool) {
    if (Object.prototype.hasOwnProperty.call(noteExpandPool, key) && key.startsWith('noteExpand')) {
      noteExpand[noteExpandKey] = noteExpandPool[key];
      if (noteExpandPool[key].active) {
        label = await tp.system.suggester(noteExpandPool[key].label , noteExpandPool[key].value, false, noteExpandPool[key].prompt);
          while (label === noteExpandPool[key].value[noteExpandPool[key].manualEntryNumber] || !label){label = await tp.system.prompt(noteExpandPool[key].input);};
        noteExpandLabel[noteExpandKey] = (label);
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
let shortDescriptionMap = defaultJson.shortDescriptionMap, shortDescriptionContent;
 while (shortDescriptionMap.active && !shortDescriptionContent) {shortDescriptionContent = await tp.system.prompt(shortDescriptionMap.prompt);};
// <!------------------------------------ Beginn Spielwiese ------------------------------------>
// <!------------------------------------- Ende Spielwiese ------------------------------------->

// <!--- Notiz Ausgabe erstellen --->
tR += `---\n`;
// <!--- Ausgabe von - Erstellungsdatum der Notiz --->
if (noteDateMap.active) {
  tR += `${noteDateMap.createdOn} ${noteDateNow}\n`
};
if (durabilityMap.active) {tR += `${durabilityMap.printValue}: ${durabilityValue}\n`;};
if (durabilityMap.active && durabilityMap.active && durabilityValue === durabilityMap.noteTemporaryValue){tR += `${durabilityDateMap.printValue}: ${durabilityDate}\n`;};

// <!--- Ausgabe von - Notiz Optionen eintragen --->
if (noteMetaPool.active){
  for (let i = 0; i < noteMeta.length; i++) {tR += `${noteMeta[i].printValue}: ${noteMetaLabel[i]}\n`};
};
// <!--- Ausgabe von - Notiz-Aliases eintragen --->
if (noteAliasMap.active){
  tR += `${defaultJson.noteAliasMap.alias}:\n`
  for (let i = 0; i < aliasTitle.length; i++) {if (aliasTitle[i] !== ""){tR += `  - ${aliasTitle[i]}\n`};};
};
// <!--- Ausgabe von - Notiz-Tags eintragen --->
if (noteTagsMap.active) {
  tR += `${noteTagsMap.tags}:
  - ${noteMainRange}\n`;
};
if (noteTagsMap.active && noteDocMetaPool.active) {
  for (let i = 0; i < noteDocMetaLabel.length; i++) {if (noteDocMetaLabel[i] !== ""){tR += `  - ${noteDocMetaLabel[i]}\n`};};
};
tR += `---\n`
if (noteTitleMap.active && docTitleMap.active || noteTitleMap.active && !docTitleMap.active) {
  tR += `# ${docTitle}\n---\n`
} else if (!noteTitleMap.active && docTitleMap.active) {
  tR += `# ${title}\n---\n`
} else {};

// <!--- Ausgabe von - Notiz-Teaser eintragen --->
if (noteSubTitleMap.active && shortDescriptionMap.active) {
  tR += `\n## ${noteSubTitleMap.shortDescriptionTitle}\n---\n${shortDescriptionContent}\n`;
} else if (!noteSubTitleMap.active && shortDescription.active) {
  tR += `\n${shortDescriptionContent}\n`;
} else if (noteSubTitleMap.active && !shortDescriptionMap.active) {
  tR += `\n## ${noteSubTitleMap.shortDescriptionTitle}\n---\n`;
} else {};
// <!------------------------------------ Beginn Spielwiese ------------------------------------>
// <!------------------------------------- Ende Spielwiese ------------------------------------->

// <!--- Ausgabe von erweiterten Informationen eintragen --->
if (noteSubTitleMap.active && noteExpandPool.active){
tR += `\n## ${noteSubTitleMap.advancedInfoTitle}\n---\n<table class=tabelle1>
<tr><th >Kategorie</th><th>Wert</th></tr>\n`;
// <!--- Ausgabe von - Notiz Optionen in die Tabelle eintragen --->
for (let i = 0; i < noteExpand.length; i++) {if (noteExpandLabel[i] === noteExpand[i].value[noteExpand[i].nonExistEntryNumber]){} else {tR += `<tr><td>${noteExpand[i].printValue}:</td><td>${noteExpandLabel[i]}</td></tr>\n`;};};
tR += `</table>\n`;
} else if (!noteSubTitleMap.active && noteExpandPool.active){
tR += `\n<table class=tabelle1>
<tr><th >Kategorie</th><th>Wert</th></tr>\n`;
// <!--- Ausgabe von - Notiz Optionen in die Tabelle eintragen --->
for (let i = 0; i < noteExpand.length; i++) {if (noteExpandLabel[i] === noteExpand[i].value[noteExpand[i].nonExistEntryNumber]){} else {tR += `<tr><td>${noteExpand[i].printValue}:</td><td>${noteExpandLabel[i]}</td></tr>\n`;};};
tR += `</table>\n`;
} else {};
if (noteSubTitleMap.active && noteSourcesMap.active){
tR += `\n## ${noteSubTitleMap.sourceTitle}\n---\n`
  // <!--- Ausgabe von - Externen Quellen und Nachweisen eintragen --->
  for (let i = 0; i < sourcesUrl.length && i < urlHttp.length && i < sourcesTitle.length; i++){
    if (sourcesUrl[i] && !urlHttp[i]){tR += `${extLink}[${sourcesTitle[i]}](${sourcesUrl[i]})&ensp;&ensp;&ensp;`
    } else {
      tR += `${extLink}[${sourcesTitle[i]}](${urlHttp[i]}${sourcesUrl[i]})&ensp;&ensp;&ensp;`
      };
  };
} else if (!noteSubTitleMap.active && noteSourcesMap.active){
  // <!--- Ausgabe von - Externen Quellen und Nachweisen eintragen --->
  for (let i = 0; i < sourcesUrl.length && i < urlHttp.length && i < sourcesTitle.length; i++){
    if (sourcesUrl[i] && !urlHttp[i]){
      tR += `${extLink}[${sourcesTitle[i]}](${sourcesUrl[i]})&ensp;&ensp;&ensp;`
      } else {tR += `${extLink}[${sourcesTitle[i]}](${urlHttp[i]}${sourcesUrl[i]})&ensp;&ensp;&ensp;`
      };
  };
} else {};

// <!------------------------------------ Beginn Spielwiese ------------------------------------>



if (noteSubTitleMap.active) {
  tR += `\n\n## ${noteSubTitleMap.internalSourceTitle}\n---\n`;
};

if (durabilityMap.internalReferencesActive) {
  tR += `[[${durabilityValue}]], `;
};

if (noteMetaPool.active && noteMetaPool.internalReferencesActive) {
  noteMetaPool.internalReferencesSelect.map(i => noteMetaLabel[i]); noteMetaPool.internalReferencesSelect.forEach(i => { tR += `[[${noteMetaLabel[i]}]], ` });
};

if (noteMainRangePool.internalReferencesActive) {
  tR += `[[${noteMainRange}]], `;
};

if (noteDocMetaPool.active && noteDocMetaPool.internalReferencesActive) {
  for (let i = 0; i < noteDocMetaLabel.length; i++) {if (noteDocMetaLabel[i] !== ""){tR += `[[${noteDocMetaLabel[i]}]], `};};
};

if (noteExpandPool.active && noteExpandPool.internalReferencesActive) {
  noteExpandPool.internalReferencesSelect.map(i => noteExpandLabel[i]);
  noteExpandPool.internalReferencesSelect.forEach(i => {
    if (noteExpandLabel[i] === noteExpand[i].value[noteExpand[i].unknownEntryNumber] || noteExpandLabel[i] === noteExpand[i].value[noteExpand[i].nonExistEntryNumber]) {
    } else {tR += `[[${noteExpandLabel[i]}]], `;}
  });
};



// <!------------------------------------- Ende Spielwiese ------------------------------------->
%>
