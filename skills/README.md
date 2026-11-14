# ContentBuilder plugin skill

A skill that teaches Claude (and ChatGPT) to build plugins for ContentBuilder and
ContentBox. Ask for one, for example *"Create a pricing table plugin with monthly and
yearly prices"*, and you get the plugin folder (`index.js`, `style.css`, `README.md`),
ready to drop into `assets/plugins/`.

## Install

**Claude app (claude.ai):** download [contentbuilder-plugin.zip](contentbuilder-plugin.zip),
then go to Settings > Capabilities > Skills and upload it.

**Claude Code:** copy the [contentbuilder-plugin](contentbuilder-plugin) folder into
`~/.claude/skills/` (for all projects) or into your project's `.claude/skills/`.

**ChatGPT:** open Skills, choose Create, then Upload from your computer, and select the zip.

## Use

Ask for a plugin. Claude picks up the skill on its own, or you can name it:
*"Use the contentbuilder-plugin skill to create a countdown timer plugin."*

Put the generated folder in `assets/plugins/` and paste the HTML from its `README.md`
into a page.
