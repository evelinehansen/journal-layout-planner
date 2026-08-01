# Journal Layout Planner

Tool to help brainstorming and visualising layout options for journalling. A sketchpad for working out how to lay out a journal page, before you commit ink to paper. The blank spread is the hard part. This lets you try arrangements, see them rendered as a page, and settle on one without wasting a real page finding out it does not work.

**[Open it here](https://evelinehansen.github.io/journal-layout-planner/)**

## What it does

- **Try layout arrangements** for a page or spread and see each one drawn out. Upload photos or use generic placeholders.
- **Preview it as a page**, so you are judging something that looks like a
  journal rather than a wireframe.
- **Change your mind freely.** Nothing is saved, so there is nothing to undo and
  nothing to tidy up afterwards.
- **Export PNG with cm ruler.** You can lock and export any layout to PNG including dimension labels and cm ruler for reference.

## Running it

Open it at
[the link above](https://evelinehansen.github.io/journal-layout-planner/). There
is nothing to install, no build step, and no account.

On an iPhone, open it in Safari and use Share, then Add to Home Screen, which
gives it its own icon.

If you clone the repo instead, the scripts are ES modules, so serve the folder
over HTTP rather than opening `index.html` from the file system:

```
python3 -m http.server 8000
```

## Where your data lives

Nowhere. This is a generator with nothing to save: it stores nothing on a server
and nothing in your browser, makes no network requests once the page has loaded,
and has no accounts, cookies, or analytics.

That means a layout lives only in the open tab. Closing or reloading the page
clears it. If you land on something you want, tap "Lock" and "Export PNG", or copy it onto
the paper page, which is where it was going anyway.

## How it's built

Plain HTML, CSS, and JavaScript. No frameworks, no build step, and no packages
pulled in from anywhere else, so the files in this repo are the whole tool: what
you can read here is what runs in your browser. There is nothing to sign in to
and no API keys or hidden configuration.

## Credits

Idea and direction by Eveline, coding by Claude. Built for my own practice,
learning and use.

Personal project, shared as is.

*This was another experiment, but I'm leaving it in public as it might be a fun/useful tool for people who are into journalling. (I'm impressed by AI, but I'm even more impressed by people who could build all this by themselves, from scratch. AI gives us non-developers just a tiny glimpse into another universe. It's quite fascinating.)*
