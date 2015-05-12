# react-list

A versatile infinite scroll [React] component.

## Install

```bash
bower install react-list

# or

npm install react-list
```

`react-list` depends on React **with addons**. `react-list` leverages the
[PureRenderMixin] to make updates more efficient.

## Examples

Check out [the test page] (and the [the test page source]) for examples of both
the `List` and `UniformList` components in action.

Here's another simple example to get you started.

```js
import React from 'react';
import {UniformList} from 'react-list';
import loadAccount from 'my-account-loader';

class MyComponent extends React.Component {
  state = {
    accounts: []
  };

  componentWillMount() {
    loadAccounts(this.handleAccounts.bind(this));
  }

  handleAccounts(accounts) {
    this.setState({accounts});
  }

  renderItem(index, key) {
    return <div key={key}>{this.state.accounts[index].name}</div>;
  }

  render() {
    return (
      <div>
        <h1>Accounts</h1>
        <div
          style={{
            position: 'relative',
            overflow: 'auto',
            maxHeight: 400
          }}
        >
          <UniformList
            itemRenderer={this.renderItem}
            length={this.state.accounts.length}
          />
        </div>
      </div>
    );
  }
}
```

## API

### `reactList.List`

The `List` component is best used when rendering items of variable heights. This
component will only attempt to draw elements once they are above the fold or
near it.

#### Props

##### initialIndex

Optionally specify an index to scroll to after mounting. For the `UniformList`,
this can be any index less than `length`. For the variable height `List`,
however, this value must be less than `pageSize`.

##### itemRenderer(index, key)

A function that receives an index and a key and returns the content to be
rendered for the item at that index.

##### itemsRenderer(items, ref)

A function that receives the rendered items and a ref. By default this element
is just a `<div>`. Generally it only needs to be overriden for use in a
`<table>` or other special case. **NOTE: You must set ref={ref} on the component
that contains the `items` so the correct item sizing calculations can be made.**

##### length

The number of items in the list.

##### pageSize (defaults to `10`)

The number of items to batch up for new renders.

##### threshold (defaults to `500`)

The number of pixels to ensure the list is buffered with.

#### Methods

##### scrollTo(index)

Put the element at `index` at the top of the viewport. **NOTE: Unlike with the
UniformList, you can only scroll to elements that have been rendered.**

---

### `reactList.UniformList`

The `UniformList` component is preferred when you can guarantee all of your
elements will be the same height. The advantage here is that the height of the
entire list can be calculated ahead of time and only enough items to fill the
viewport ever need to be drawn.

`UniformList` has the same properties as `List`, but `pageSize` is calculated
automatically. It also supports two more optional properties.

#### Props

##### itemHeight (defaults to automatic calculation)

While not necessary, providing the exact rendered height of each item can
improve performance. If not provided, the height of the first rendered element
will be used to assume the height of all items.

##### itemsPerRow (defaults to automatic calculation)

Similar to `itemHeight`, providing the number of items that render per row of
items (columns) can improve performance. This is also not required and if not
provided will be calculated by counting the number of rendered items in the
first row.

**NOTE: `itemHeight` and `itemsPerRow` should always both be defined or both not
be defined. If one is defined but not the other, the automatic calculation will
take over.**

#### Methods

##### scrollTo(index)

Put the element at `index` at the top of the viewport.

##### scrollAround(index)

Scroll the viewport so that the element at `index` is visible.

---

### `reactList.QuantizedHeightList`

A list where all item heights are a multiple of some base size and the
data model is assumed to be asynchronous and capable of mapping positions from
this arbitrary unit space to a list of items.  Multiple items per row are not
supported.

For example, in an email client you might want to display already-read messages
as a single unit in height but display unread messages as three units high so
that there is space to include a large excerpt of the email.

The flow of operations in response to user action works like this:
- The user scrolls, QuantizedHeightList processes the change
- If the position has changed by more than `seekThreshold` pixels since the last
  seek, `props.seek` method that must be provided by the parent component is
  invoked.
- The parent's `seek` method does whatever is required, re-rendering itself and
  passing updated props values for `seekedData` and `seekedOffset`.
- `itemRenderer` is called for all of the objects in `seekedData`

From the perspective of the parent component, it is always passing a continuous
list of variable-sized content to be displayed.  It provides `seek` for the
`QuantizedHeightList` to ensure that this display area overlaps with what the
user is looking at, but the `QuantizedHeightList` is not actually dictating
anything, merely requesting it.

#### Props

##### itemRenderer(seekedDataItem, relativeIndex, unitSize)

A function called once for each object in `seekedData` and produces the
appropriately sized content for the value.  The returned content should include
a key for efficiency purposes and you can use `relativeIndex` if you have no
better identifier for keying purposes.

`unitSize` is provided as a handy reminder of the sizing multiple for your
returned content.  Unless you can ensure that the layout engine will result in
the appropriate size from other properties, you probably want to explicitly
style the returned content to have an explicit height and "overflow: hidden".
If this sizing invariant is not maintained,

##### seek(offsetUnits, bufferUnitsAbove, visibleUnits, bufferUnitsBelow)

A function to be provided by the parent component to initiate a seek.  This
should eventually result in the `seekedData` and `seekedOffset` props of the
`QuantizedHeightList` being updated if the seek request results in meaningful
changes.

The arguments very explicitly break out what is for threshold buffering and what
is intended for display so that the data model can prioritize any expensive
actions.  For example, an email application that fetches snippets on demand
would want to prioritize for what the user can currently see, not waste energy
fetching data for something the user is not looking at.

The arguments are:
- `offsetUnits`: The offset in the quantized height units for the very first
  item we want data for.
- `bufferUnitsAbove`: The number of quantized height units desired for buffering
  above the currently visible area.  This may be less than `threshold` if near
  the top of the list.
- `visibleUnits`: The number of quantized units desired for immediate display.
  Partially visible pieces of a conceptual single-unit item are fully included
  here.
- `bufferUnitsBelow`: The number of quantized units desired for buffering below
  the currently visible area.

##### seekedData

An list of opaque objects to be rendered by `itemRenderer`.  `itemRenderer` will
be called with each item.  In other words, we do not impose any requirements on
the contents of the data.  You just need to make sure `itemRenderer` has all the
information it needs.

##### seekedOffset

The position of the first item in `seekedData` in the arbitrary units used by
the list.

For example, if the parent receives a call to seek to an offset of 4, but the
first two items both have size 3, then `seekedOffset` would have a value of 3,
since that is the first known item that covers offset 4.

##### seekThreshold

The minimum number of CSS pixels that the user must scroll before a new call to
`seek` will be issued since the last call was issued.

##### totalHeight

The height of the scrollable area in quantized units.

##### unitSize

The number of CSS pixels that an item of quantized size 1 will occupy.


[React]: https://github.com/facebook/react
[PureRenderMixin]: https://facebook.github.io/react/docs/pure-render-mixin.html
[the test page]: https://orgsync.github.io/react-list/
[the test page source]: index.html
