# Fixes Strategies

Differents stategies can be identified for the fixes. Whatever the fix, we want to ensure:

- Searching in the notetbook functionality is still working like before.
- No negative side-effect should appear.

## Strip Output

We could strip the cell output if too the ouput is too large, but this has two limits:

1. This does not work for graphical outputs.
2. This does not provide any fix for the code editor which is the largest identified performance offender so far.

## Block Div

We should not use `flex` display CSS layout but rather use a simple `block`.

##  Virtualized Notebook

Cocalc has been [using react-virtualized](https://github.com/sagemathinc/cocalc/pull/3969). We should look at this to understand how this could help Virtualization complexity and potential side-effects (search...) have to be taken into account.

### Intersection Observer

We can think to more generic fixes like using [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API).

- [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)
- [Intersection Observer API - Timing](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API/Timing_element_visibility)

The strategy for a notebook would be:

- Use an intersection observer to only render cells (input + output) on screen on page load.
- As there are free cycles, render the rest of the notebook.
- Ideally do this from the closest cell to the furthest.
- If any cell is in view, it loads.
- All non-loaded cells will need a cheap div that takes approx the same amount of space and has clear loading text.
- Add UI elements to indicate a notebook is still loading so that users don't search and not find something on the page.
- Do not load any notebook that is not visible (i.e. when you open a workspace and have notebooks that are not visible, they should not be rendered).

Tutorials

- <https://webdesign.tutsplus.com/tutorials/how-to-intersection-observer--cms-30250>
- <https://css-tricks.com/a-few-functional-uses-for-intersection-observer-to-know-when-an-element-is-in-view>

React

An preliminary step is to wrap Notebook into React (see this PR [Try Notebok React component](https://github.com/jupyterlab/benchmarks/issues/15)).

Then we could use React Intersection Obsever libraries.

- [React Intersection Observer (thebuilder)](https://github.com/thebuilder/react-intersection-observer) ([storybook](https://react-intersection-observer.now.sh))
- [React Intersection Observer (researchgate)](https://github.com/researchgate/react-intersection-observer) ([storybook](https://researchgate.github.io/react-intersection-observer))
- [React Cool Inview](https://github.com/wellyshen/react-cool-inview) ([example](https://react-cool-inview.netlify.app))
- [react-infinite-grid-scroller](https://github.com/HenrikBechmann/react-infinite-grid-scroller) (horizontal and vertical scrolling)

### React Virtualized / Windowing

An preliminary step is to wrap Notebook into React (see this PR [Try Notebok React component](https://github.com/jupyterlab/benchmarks/issues/15)).

Then we could use React virtualisation libraries.

- [Creating More Efficient React Views with Windowing - ForwardJS San Francisco](https://www.youtube.com/watch?v=t4tuhg7b50I)
- [Rendering large lists with React Virtualized](https://www.youtube.com/watch?v=UrgfPjX97Yg)

- [React Virtualized](https://github.com/bvaughn/react-virtualized)
- [React Window](https://github.com/bvaughn/react-window)

- [Rendering large lists with react-virtualized or react-window](https://www.youtube.com/watch?v=QhPn6hLGljU)
- <https://addyosmani.com/blog/react-window>
- <https://github.com/falinsky/tmdb-viewer> (react-virtualized) https://tmdb-viewer.surge.sh
- <https://github.com/giovanni0918/tmdb-viewer> (react-window)

- [React Virtual](https://github.com/tannerlinsley/react-virtual)

## Shadow DOM

[Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM) allows `encapsulation — being able to keep the markup structure, style, and behavior hidden and separate from other code on the page so that different parts do not clash, and the code can be kept nice and clean`.

Initial Shadow DOM usage is implemented in [Move CodeMirror HTML tree and related CSS to shadow DOM](https://github.com/jupyterlab/jupyterlab/pull/8584 . We compare `1f15fcb` vs `f7b7ee7` commits.

```
- f7b7ee7d271bd1233a5b95c9fd9dfb2d9509bbe6
  - Move CodeMirror HTML tree and related CSS to shadow DOM
  - Sun Jul 26 05:33:10 2020 -0500ƒ
- 1f15fcbc577517f1f320252bbe0a7b5a48f32996
  - Merge pull request #8642 from saulshanabrook/2.2-changelog
  - Fri Jul 24 16:14:11 2020 -0400
```

For the `100 N output each of a div` and the `one output with N div`, we find that Shadow DOM made switching notebooks slightly faster in Chrome and slightly slower in Firefox.

![](images/shadow-dom/f7b7ee7d271bd1233a5b95c9fd9dfb2d9509bbe6.png "")

For the `Nx5 with 2 line of code and 2 outputs per cell`, we have some small improvements in notebook switching. For Firefox open times do see nice improvement in general. On Chrome it seems also improve for smaller notebooks but it degrades with larger ones, and actually becomes worse (with a lot of variability).

![](images/shadow-dom/91180448-5a7b9280-e6ad-11ea-8e33-5ffa780126cf.png "")

## Tune CodeMirror Configuration

We should look how to configure or even update CodeMirror code base to mitigate the numerous Force layout.

## Enhance CodeMirror Code Base

On this [comment](https://github.com/jupyterlab/jupyterlab/issues/4292#issuecomment-674419945): Also editing the codemirror source to avoid measurements (by manually returning what the cached values ended up being) at <https://github.com/codemirror/CodeMirror/blob/83b9f82f411274407755f80f403a48448faf81d0/src/measurement/position_measurement.js#L586> and <https://github.com/codemirror/CodeMirror/blob/83b9f82f411274407755f80f403a48448faf81d0/src/measurement/position_measurement.js#L606> seemed to help a bit. The idea here is that since a single codemirror seems okay, but many codemirrors does not (even when the total number of lines is the same), perhaps we can use measurements from the codemirror to shortcut measurements in all the others, which seem to be causing lots of browser layout time.

Read also the discussion on [CodeMirror/#/5873](https://github.com/codemirror/CodeMirror/issues/5873).

## Content Visibility

We have tried [content visibility](https://web.dev/content-visibility) supported in Chrome 85+ in [this branch](https://github.com/datalayer-contrib/jupyterlab/tree/2.2.x-content-visibility).

Without Shadow DOM, we have clear improvement.

![](images/content-visibility/content-visiblity-no-shadow-dom.png "")

Meanwhile, with Shadow DOM, we have no improvement.

See also [Display Locking library](https://github.com/wicg/display-locking) for the related `Display Locking` specification.

## Fast DOM

[FastDOM](https://github.com/wilsonpage/fastdom) is a library that eliminates layout thrashing by batching DOM measurement and mutation tasks.
 
## Web Workers

[Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) makes it possible to run a script operation in a background thread separate from the main execution thread of a web application. The advantage of this is that laborious processing can be performed in a separate thread, allowing the main (usually the UI) thread to run without being blocked/slowed down.

- [Web Workers API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API)
- [React and Webworkers](https://github.com/facebook/react/issues/3092#issuecomment-333417970)

## Service Workers

- [Service Workers](https://developers.google.com/web/fundamentals/primers/service-workers)

## Use requestAnimationFrame (non concluding)

Wrapped the cell creation into a requestAnimationFrame call.

```javascript
requestAnimationFrame(() => {
  const cellDB = this._factory.modelDB!;
  const cellType = cellDB.createValue(id + '.type');
  let cell: ICellModel;
  switch (cellType.get()) {
    case 'code':
    cell = this._factory.createCodeCell({ id: id });
    break;
    case 'markdown':
    cell = this._factory.createMarkdownCell({ id: id });
    break;
    default:
    cell = this._factory.createRawCell({ id: id });
    break;
  }
  this._cellMap.set(id, cell);
});
```

This produces a different profile pattern (2 heavy sections separated by an inactive one). The Forced layout due to codemirror are still there.

![](images/profiles/89731086-9d4e3100-da44-11ea-9e2b-292f8a14920c.png "")

## Update Editor on Show

A candidate fix would be to further look at `updateEditorOnShow` implemented in [jupyterlab/jupyterlab#5700](https://github.com/jupyterlab/jupyterlab/issues/5700), but [is is already set to false...](https://github.com/jupyterlab/jupyterlab/blob/71f07379b184d5b0b8b67b55163d27194a61a0ac/packages/notebook/src/widget.ts#L493).

## Use pushAll cells insted of push (non concluding)

We have updated the [celllist#pushAll](https://github.com/jupyterlab/jupyterlab/blob/7d1e17381d3ed61c23c189822810e8b4918d57ba/packages/notebook/src/celllist.ts#L333-L341) code block but it has not brought better performance.

Current attempts have not brought enhancements.

## React Concurrency

We can inject more React into the UI components and see if it makes life easier to get better performance using core React performance features.

- For React components, [React Concurrency](https://reactjs.org/docs/concurrent-mode-intro.html#concurrency) can be used.
- [Try Notebok React component](https://github.com/jupyterlab/benchmarks/issues/15) to wrap the outputs in React.

## Reuse contentFactory

Reuse contentFactory in Notebook Model. We may try to benchmark with this patch. At first user try, this does not give sensible change.

```
diff --git a/packages/notebook/src/model.ts b/packages/notebook/src/model.ts
index 2efeee7b3..4716ce9f7 100644
--- a/packages/notebook/src/model.ts
+++ b/packages/notebook/src/model.ts
@@ -514,7 +514,7 @@ export namespace NotebookModel {
      *   `codeCellContentFactory` will be used.
      */
     createCodeCell(options: CodeCellModel.IOptions): ICodeCellModel {
-      if (options.contentFactory) {
+      if (!options.contentFactory) {
         options.contentFactory = this.codeCellContentFactory;
       }
       if (this.modelDB) {
```

## Use scrollbarStyle: 'null' in Editor Config

On this [comment](https://github.com/jupyterlab/jupyterlab/issues/4292#issuecomment-674419945): I did some quick experiments, based on some quick profiling results (it seems that the vast majority of time is in browser layout). For example, adding scrollbarStyle: 'null' to the bare editor config in [editor.ts](https://github.com/jupyterlab/jupyterlab/blob/7d1e17381d3ed61c23c189822810e8b4918d57ba/packages/codemirror/src/editor.ts#L1374).

## Transfer Content in Chunks

Transfer Content in Chunks for Incremental Loading on <https://github.com/jupyter/jupyter_server/issues/308>

## Server Memory cache

TBD

## Improving Network Performance

- [Improving Network Performance](https://github.com/jupyter/jupyter_server/issues/312).

## Web Render

From this [comment](https://github.com/jupyterlab/jupyterlab/issues/4292#issuecomment-674411129): Webrender for Firefox 79 (for many linux and macos devices, see <https://wiki.mozilla.org/Platform/GFX/WebRender_Where>  can be turned on via a pref. See also <https://www.techrepublic.com/article/how-to-enable-firefox-webrender-for-faster-page-rendering>. Note that windows firefox has had webrender turned on by default in certain cases for a while now.

