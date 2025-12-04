<%* await tp.file.include("[[./data/config]]");
'use strict';
// Note selection, JSON parsing, and creating basic data types
const defaultJsonPath = '' + templatePath + jsonPath;
const fs = require('fs');
const defaultJson = JSON.parse( fs.readFileSync(defaultJsonPath, 'utf8'));
const noteMainRangePool = defaultJson.noteMainRangePool
let noteMainRange = await tp.system.suggester(noteMainRangePool.label, noteMainRangePool.value, false, noteMainRangePool.prompt);
const noteMainRangeSelected = noteMainRangePool.selection[noteMainRange];
const noteMainRangeJson = JSON.parse ( fs.readFileSync(templatePath + noteMainRangeSelected.noteMainRangeValue, 'utf8'));
const noteTagsMap = defaultJson.noteTagsMap;
const noteBasicMap = defaultJson.noteBasicMap;
const noteSubTitleMap = defaultJson.noteSubTitleMap;
const docIconMap = noteMainRangeJson.docIconMap;
let key = 0;

// Determine note date
const noteDateMap = defaultJson.noteDateMap;
const noteDateNow = tp.date.now(noteBasicMap.dateFormat);

// Note Set validity
const durabilityMap = defaultJson.durabilityMap;
let durabilityLabel = [durabilityMap.notePermanentlyLabel, durabilityMap.noteTemporaryLabel];
let durabilityValue = [durabilityMap.notePermanentlyValue, durabilityMap.noteTemporaryValue];
if (durabilityMap.active) {
  label = await tp.system.suggester(durabilityLabel, durabilityValue, false, durabilityMap.prompt);
};
durabilityValue = label;

// If validity is limited, set an end date
const durabilityDateMap = defaultJson.durabilityDateMap;
const durabilitySignMap = defaultJson.durabilitySignMap;
if (durabilityValue === durabilityMap.noteTemporaryValue) {
	durabilityDate = await tp.system.prompt(durabilityDateMap.prompt, tp.date.now(noteBasicMap.dateFormat, "P3M"));
	while (!durabilityDate){
    durabilityDate = await tp.system.prompt(durabilityDateMap.prompt, tp.date.now(noteBasicMap.dateFormat, "P3M"));
    };
	noteSign = durabilitySignMap.noteSignTemporary;
} else {
  noteSign = durabilitySignMap.noteSignPermanently
};

// Set note name and create note
const docTitleMap = defaultJson.docTitleMap;
const space = noteBasicMap.space;
const mark = docIconMap.mark
let title = tp.file.title;
while (title.startsWith("Untitled") || !title){title = await tp.system.prompt(docTitleMap.titleFieldInsert);};

if (docIconMap.active && durabilitySignMap.active){
  await tp.file.rename(mark  + space + noteSign  + space + title);
} else if (docIconMap.active && !durabilitySignMap.active) {
  await tp.file.rename(mark + space + title);
} else if (!docIconMap.active && durabilitySignMap.active) {
  await tp.file.rename(noteSign + space + title);
} else {
  await tp.file.rename(title);
};

// Set note title
const noteTitleMap = defaultJson.noteTitleMap;
let docTitle;
if (noteTitleMap.active) {
  while (!docTitle){docTitle = await tp.system.prompt(noteTitleMap.docTitleFieldInsert);};
};

// Set note aliases
const noteAliasMap = defaultJson.noteAliasMap;
let aliasTitle = [];
if (noteAliasMap.active) {
  while (!aliasTitle[key]){
    aliasTitle[key] = await tp.system.prompt(noteAliasMap.aliasTitleFieldInsert0);
    };
  while (aliasTitle[key]){
    key++; aliasTitle[key] = await tp.system.prompt(noteAliasMap.aliasTitleFieldInsert1)
  };
};

// Add note to MetaPool
const noteMetaPool = defaultJson.noteMetaPool;
let noteMeta = [], noteMetaLabel = [], noteMetaUnknown = [], noteMetaNonExist = [], noteMetaKey = 0;
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
        noteMetaUnknown[noteMetaKey] = noteMetaPool[key].value[noteMetaPool[key].unknownEntryNumber];
        noteMetaNonExist[noteMetaKey] = (noteMetaPool[key].value[noteMetaPool[key].nonExistEntryNumber]);
        noteMetaKey++;
      };
    };
  };
};

// Enter MetaDodPool note
const noteMetaDocPool = noteMainRangeJson.noteMetaDocPool;
let noteMetaDoc = [], noteMetaDocLabel = [], noteMetaDocUnknown = [], noteMetaDocNonExist = [], noteMetaDocKey = 0;
if (noteMetaDocPool.active) {
  for (const key in noteMetaDocPool) {
    if (Object.prototype.hasOwnProperty.call(noteMetaDocPool, key) && key.startsWith('noteMetaDoc')) {
      noteMetaDoc[noteMetaDocKey] = noteMetaDocPool[key]
      if (noteMetaDocPool[key].active) {
        label = await tp.system.suggester(noteMetaDocPool[key].label , noteMetaDocPool[key].value, false, noteMetaDocPool[key].prompt);
        while (label === noteMetaDocPool[key].value[noteMetaDocPool[key].manualEntryNumber] || !label){
          label = await tp.system.prompt(noteMetaDocPool[key].input);
        };
        noteMetaDocLabel[noteMetaDocKey] = (label);
        noteMetaDocUnknown[noteMetaDocKey] = noteMetaDocPool[key].value[noteMetaDocPool[key].unknownEntryNumber];
        noteMetaDocNonExist[noteMetaDocKey] = (noteMetaDocPool[key].value[noteMetaDocPool[key].nonExistEntryNumber]);
        noteMetaDocKey++;
      };
    };
  };
};

// Create document category and subcategory
const noteMetaDocTagPool = noteMainRangeJson.noteMetaDocTagPool
let noteMetaDocTag = [], noteMetaDocTagLabel = [], noteMetaDocTagUnknown = [],noteMetaDocTagNonExist = [], noteMetaDocTagKey = 0;
if (noteMetaDocTagPool.active) {
  for (const key in noteMetaDocTagPool) {
    if (Object.prototype.hasOwnProperty.call(noteMetaDocTagPool, key) && key.startsWith('noteMetaDocTag')) {
      noteMetaDocTag[noteMetaDocTagKey] = noteMetaDocTagPool[key];
      if (noteMetaDocTagPool[key].active) {
        label = await tp.system.suggester(noteMetaDocTagPool[key].label , noteMetaDocTagPool[key].value, false, noteMetaDocTagPool[key].prompt);
        if (label === noteMetaDocTagPool[key].value[noteMetaDocTagPool[key].manualEntryNumber]) {
          label = await tp.system.prompt(noteMetaDocTagPool[key].input);
          while (!label){label = await tp.system.prompt(noteMetaDocTagPool[key].input);};
        };
        noteMetaDocTagLabel[noteMetaDocTagKey] = (label);
        noteMetaDocTagUnknown[noteMetaDocTagKey] = noteMetaDocTagPool[key].value[noteMetaDocTagPool[key].unknownEntryNumber];
        noteMetaDocTagNonExist[noteMetaDocTagKey] = (noteMetaDocTagPool[key].value[noteMetaDocTagPool[key].nonExistEntryNumber]);
        noteMetaDocTagKey++;
      };
      let selected = noteMainRangeJson.noteMetaDocTagPool[key].second[label];
      let manualEntry = noteMetaDocTagPool[key].value[noteMetaDocTagPool[key].manualEntryNumber]
      if (selected && noteMetaDocTagPool[key].second.active && selected !== manualEntry) {
        label = await tp.system.suggester(selected.label, selected.value, false, selected.prompt);
         if (label === noteMetaDocTagPool[key].value[noteMetaDocTagPool[key].manualEntryNumber]) {
          label = await tp.system.prompt(selected.input);
          while (!label){label = await tp.system.prompt(selected.input);};
        };
        noteMetaDocTagLabel[noteMetaDocTagKey] = (label);
        noteMetaDocTagUnknown[noteMetaDocTagKey] = noteMetaDocTagPool[key].value[noteMetaDocTagPool[key].unknownEntryNumber];
        noteMetaDocTagNonExist[noteMetaDocTagKey] = (noteMetaDocTagPool[key].value[noteMetaDocTagPool[key].nonExistEntryNumber]);
      };
    };
  };
};

// Create advanced note information
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

// Create external sources and references
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

// Create note teaser
const shortDescriptionMap = defaultJson.shortDescriptionMap;
let shortDescriptionContent;
while (shortDescriptionMap.active && !shortDescriptionContent) {
  shortDescriptionContent = await tp.system.prompt(shortDescriptionMap.prompt);
  };

// Write down expenses
tR += `---\n\n`;
// Write the note creation date
if (noteDateMap.active) {
  tR += `${noteDateMap.createdOn} ${noteDateNow}\n`
};
if (durabilityMap.active) {tR += `${durabilityMap.printValue}: ${durabilityValue}\n`;};
if (durabilityMap.active && durabilityMap.active && durabilityValue === durabilityMap.noteTemporaryValue){
  tR += `${durabilityDateMap.printValue}: ${durabilityDate}\n`;
};

// Notiz Optionen schreiben
if (noteMetaPool.active){
  for (let i = 0; i < noteMeta.length; i++) {
    if (noteMetaLabel[i] !== noteMetaNonExist[i]){tR += `${noteMeta[i].printValue}: ${noteMetaLabel[i]}\n`};
  };
};

// Notiz Optionen schreiben
if (noteMetaDocPool.active){
  for (let i = 0; i < noteMetaDoc.length; i++) {
    if (noteMetaDocLabel[i] !== noteMetaDocNonExist[i]){tR += `${noteMetaDoc[i].printValue}: ${noteMetaDocLabel[i]}\n`};
  };
};

// Write note aliases
if (noteAliasMap.active){
  tR += `${defaultJson.noteAliasMap.alias}:\n`
  for (let i = 0; i < aliasTitle.length; i++) {if (aliasTitle[i] !== ""){tR += `  - ${aliasTitle[i]}\n`};};
};

// Write note tags
if (noteTagsMap.active) {
  tR += `${noteTagsMap.tags}:
  - ${noteMainRange}\n`;
};
if (noteTagsMap.active && noteMetaDocTagPool.active) {
  for (let i = 0; i < noteMetaDocTagLabel.length; i++) {
    if (noteMetaDocTagLabel[i] !== noteMetaDocTagNonExist[i]){tR += `  - ${noteMetaDocTagLabel[i]}\n`};
  };
};
tR += `---\n\n`
if (noteTitleMap.active && docTitleMap.active || noteTitleMap.active && !docTitleMap.active) {
  tR += `# ${docTitle}\n---\n\n`
} else if (!noteTitleMap.active && docTitleMap.active) {
  tR += `# ${title}\n---\n\n`
};

// Write subtitles and note teasers
if (noteSubTitleMap.active && shortDescriptionMap.active) {
  tR += `\n## ${noteSubTitleMap.shortDescriptionTitle}\n---\n\n${shortDescriptionContent}\n`;
} else if (!noteSubTitleMap.active && shortDescription.active) {
  tR += `\n${shortDescriptionContent}\n`;
} else if (noteSubTitleMap.active && !shortDescriptionMap.active) {
  tR += `\n## ${noteSubTitleMap.shortDescriptionTitle}\n---\n\n`;
};

// Write subtitles and notes Sample
const sampleContentPool = noteMainRangeJson.sampleContentPool;
let sampleContent = [], sampleContentKey = 0;
if (noteSubTitleMap.active && sampleContentPool.active){
  tR += `\n## ${noteSubTitleMap.detailDescriptionTitle}\n---\n\n`;
  // Write a sample note
  for (const key in sampleContentPool) {
    if (Object.prototype.hasOwnProperty.call(sampleContentPool, key) && key.startsWith('sampleContent') && sampleContentPool[key].active ) {
      tR += `${sampleContentPool[key].value}`;
    };
  };
} else if (!noteSubTitleMap.active && sampleContentPool.active){
  // Write a sample note
  if (Object.prototype.hasOwnProperty.call(sampleContentPool, key) && key.startsWith('sampleContent') && sampleContentPool[key].active ) {
    tR += `${sampleContentPool[key].value}`;
  };
} else if (noteSubTitleMap.active && !sampleContentPool.active){
  tR += `\n## ${noteSubTitleMap.detailDescriptionTitle}\n---\n\n`;
};

// Write subtitles and additional information
if (noteSubTitleMap.active && noteExpandPool.active){
  // Write subtitles and tables Head
  tR += `\n## ${noteSubTitleMap.advancedInfoTitle}\n---\n\n| ${noteExpandPool.tableFirstRowTitle} | ${noteExpandPool.tableSecondRowTitle} |\n|---|---|\n`;
  // Write note options in the table
  for (let i = 0; i < noteExpand.length; i++) {if (noteExpandLabel[i] !== noteExpandNonExist[i]){
    tR += `| ${noteExpand[i].printValue}: | ${noteExpandLabel[i]} |\n`;};
  };
} else if (!noteSubTitleMap.active && noteExpandPool.active){
  tR += `\n| ${noteExpandPool.tableFirstRowTitle} | ${noteExpandPool.tableSecondRowTitle} |\n|---|---|\n`;
  // Write note options in the table
  for (let i = 0; i < noteExpand.length; i++) {if (noteExpandLabel[i] !== noteExpandNonExist[i]){
    tR += `| ${noteExpand[i].printValue}: | ${noteExpandLabel[i]} |\n`;};
  };
} else if (noteSubTitleMap.active && !noteExpandPool.active){
  tR += `\n## ${noteSubTitleMap.advancedInfoTitle}\n---\n\n`;
};

// Write subtitles and external sources and references
if (noteSubTitleMap.active && noteSourcesMap.active){
  tR += `\n## ${noteSubTitleMap.sourceTitle}\n---\n\n`
  // External sources and references writing
  for (let i = 0; i < sourcesUrl.length && i < sourcesTitle.length; i++){
    if (sourcesUrl[i] && !urlHttp[i]){
      tR += `${extLink}[${sourcesTitle[i]}](${sourcesUrl[i]})\n`
    } else {
      tR += `${extLink}[${sourcesTitle[i]}](${urlHttp[i]}${sourcesUrl[i]})\n`
    };
  };
} else if (!noteSubTitleMap.active && noteSourcesMap.active){
  // External sources and references writing
  for (let i = 0; i < sourcesUrl.length && i < sourcesTitle.length; i++){
    if (sourcesUrl[i] && !urlHttp[i]){
      tR += `${extLink}[${sourcesTitle[i]}](${sourcesUrl[i]})\n`
    } else {
        tR += `${extLink}[${sourcesTitle[i]}](${urlHttp[i]}${sourcesUrl[i]})\n`
    };
  };
} else if (noteSubTitleMap.active && !noteSourcesMap.active){
  tR += `\n## ${noteSubTitleMap.sourceTitle}\n---\n\n`;
};

// Writing subtitles and internal references
const intRefDefault = defaultJson.internalReferencesMap;
const intRefMainRange = noteMainRangeJson.internalReferencesMap;
const contentMark = noteBasicMap.contentMark
if (noteSubTitleMap.active && intRefDefault.active || intRefMainRange.active) {
  // Write subtitles
  tR += `\n\n## ${noteSubTitleMap.internalSourceTitle}\n---\n\n`;
  // Write internal references
  if (durabilityMap.active && intRefDefault.active && intRefDefault.durability.active) {
    tR += `[[${contentMark} ${durabilityValue}]], `;
    if (noteMetaPool.active && intRefDefault.active && intRefDefault.noteMeta.active) {
      intRefDefault.noteMeta.select.map(i => noteMetaLabel[i]);
      intRefDefault.noteMeta.select.forEach(i => { if (noteMetaLabel[i] !== noteMetaUnknown[i] && noteMetaLabel[i] !== noteMetaNonExist[i]) {tR += `[[${contentMark} ${noteMetaLabel[i]}]], `;} });
    };
    if (intRefDefault.active) {
      tR += `[[${contentMark} ${noteMainRange}]], `;
    };
    if (noteMetaDocPool.active && intRefMainRange.active && intRefMainRange.noteMetaDoc.active) {
      intRefMainRange.noteMetaDoc.select.map(i => noteMetaDocLabel[i]);
      intRefMainRange.noteMetaDoc.select.forEach(i => {if (noteMetaDocLabel[i] !== noteMetaDocUnknown[i] && noteMetaDocLabel[i] !== noteMetaDocNonExist[i]) {tR += `[[${contentMark} ${noteMetaDocLabel[i]}]], `;} });
    };
    if (noteMetaDocTagPool.active && intRefMainRange.active && intRefMainRange.noteMetaDocTag.active) {
      intRefMainRange.noteMetaDocTag.select.map(i => noteMetaDocTagLabel[i]);
      intRefMainRange.noteMetaDocTag.select.forEach(i => {if (noteMetaDocTagLabel[i] !== noteMetaDocTagUnknown[i] && noteMetaDocTagLabel[i] !== noteMetaDocTagNonExist[i]) {tR += `[[${contentMark} ${noteMetaDocTagLabel[i]}]], `;} });
    };
    if (noteExpandPool.active && intRefMainRange.active && intRefMainRange.noteExpand.active) {
      intRefMainRange.noteExpand.select.map(i => noteExpandLabel[i]);
      intRefMainRange.noteExpand.select.forEach(i => {if (noteExpandLabel[i] !== noteExpandUnknown[i] && noteExpandLabel[i] !== noteExpandNonExist[i]) {tR += `[[${contentMark} ${noteExpandLabel[i]}]], `;} });
    };
  };
} else if (!noteSubTitleMap.active && intRefDefault.active || intRefMainRange.active) {
  // Write internal references
  if (durabilityMap.active && intRefDefault.active && intRefDefault.durability.active) {
    tR += `[[${contentMark} ${durabilityValue}]], `;
    if (noteMetaPool.active && intRefDefault.active && intRefDefault.noteMeta.active) {
      intRefDefault.noteMeta.select.map(i => noteMetaLabel[i]);
      intRefDefault.noteMeta.select.forEach(i => { if (noteMetaLabel[i] !== noteMetaUnknown[i] && noteMetaLabel[i] !== noteMetaNonExist[i]) {tR += `[[${contentMark} ${noteMetaLabel[i]}]], `;} });
    };
    if (intRefDefault.active) {
      tR += `[[${contentMark} ${noteMainRange}]], `;
    };
    if (noteMetaDocPool.active && intRefMainRange.active && intRefMainRange.noteMetaDoc.active) {
      intRefMainRange.noteMetaDoc.select.map(i => noteMetaDocLabel[i]);
      intRefMainRange.noteMetaDoc.select.forEach(i => {if (noteMetaDocLabel[i] !== noteMetaDocUnknown[i] && noteMetaDocLabel[i] !== noteMetaDocNonExist[i]) {tR += `[[${contentMark} ${noteMetaDocLabel[i]}]], `;} });
    };s
    if (noteMetaDocTagPool.active && intRefMainRange.active && intRefMainRange.noteMetaDocTag.active) {
      intRefMainRange.noteMetaDocTag.select.map(i => noteMetaDocTagLabel[i]);
      intRefMainRange.noteMetaDocTag.select.forEach(i => {if (noteMetaDocTagLabel[i] !== noteMetaDocTagUnknown[i] && noteMetaDocTagLabel[i] !== noteMetaDocTagNonExist[i]) {tR += `[[${contentMark} ${noteMetaDocTagLabel[i]}]], `;} });
    };
    if (noteExpandPool.active && intRefMainRange.active && intRefMainRange.noteExpand.active) {
      intRefMainRange.noteExpand.select.map(i => noteExpandLabel[i]);
      intRefMainRange.noteExpand.select.forEach(i => {if (noteExpandLabel[i] !== noteExpandUnknown[i] && noteExpandLabel[i] !== noteExpandNonExist[i]) {tR += `[[${contentMark} ${noteExpandLabel[i]}]], `;} });
    };
  };
} else if (noteSubTitleMap.active && !intRefDefault.active && !intRefMainRange.active) {
  tR += `\n\n## ${noteSubTitleMap.internalSourceTitle}\n---\n\n`;
};
%>
