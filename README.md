# stage0

Collection of low-level DOM tools for building high performant web interfaces.

## Eh?

Given `h` function for extracting DOM references, organize work whatever you like and use full power of native DOM API.

## How can I use it?

```javascript
import h from 'stage0'

// Let's build some Component
const itemView = h`
  <tr>
      <td class="col-md-1">#id</td>
      <td class="col-md-4">
          <a #select>#label</a>
      </td>
      <td class="col-md-1"><a #del><span class="glyphicon glyphicon-remove" aria-hidden="true"></span></a></td>
      <td class="col-md-6"></td>
  </tr>
`
function Item(item, scope) {
  const root = itemView

  // Collect references to dynamic parts
  const refs = itemView.collect(root)

  const {id, label, select, del} = refs

  // One time data binding
  id.nodeValue = item.id
  label.nodeValue = item.label
  select.onclick = () => scope.select(item)
  del.onclick = () => scope.del(item)

  // Handcrafted update function, we know exactly what parts of component will change after creation
  // and what parameters we need to update the view
  let a = '', a2,
      b = item.label, b2
  root.update = function(selected) {
    a2 = item.id === selected ? 'danger' : ''
    b2 = item.label
    
    if (a2 !== a) a = root.className = a2
    if (b2 !== b) b = label.nodeValue = b2
  }

  return root
}

// Create component
const node = Item({id: 1, label: 'Wow'}, {
    select: item => console.debug({item}),
    del: item => console.debug({item})
})
document.body.appendChild(node)

// And update the node
const selected = 1
node.update(selected)
```

## h
```javascript
import h from 'stage0'

const node = h`
    <div #root>
        <span>#header</span>
        <div #content></div>
    </div>
`
// will give you ready to use DOM node, which you can clone or append directly wherever you need

// h generates function with memoized DOM paths for obtaining references.
// Given #-syntax, you can collect references to DOM nodes in this DOM 
// or cloned or any other DOM node with similar structure.

const refs = node.collect(node)
// refs === {root: Node, header: Node, content: Node}
```

## setupSyntheticEvent
```javascript
import {setupSyntheticEvent} from 'stage0'

setupSyntheticEvent('click')
// will setup global event handler, that will run handler from nearest predecessor in DOM tree

// To attach event handler, simply do
node.__click = () => console.debug('click')
```

## reconcile
```javascript
import reconcile from 'stage0/reconcile'

// Reconcile nodes in given parent, according to new and previous rendered data arrays
// Used for displaying node arrays
reconcile(
    parent,
    renderedValues,
    newValues,
    // Create callback
    item => document.createTextNode(item),
    // Update callback
    (node, item) => node.nodeValue = item + ' !!!'
)
```

## reuseNodes
```javascript
import reuseNodes from 'stage0/reuseNodes'

// Similar to reconcile, with exception that it will not move any node, 
// doing only updates on all nodes and adding/removing nodes if neccessary
// Used as more performant alternative of reconcile
reuseNodes(
    parent,
    renderedValues,
    newValues,
    // Create callback
    item => document.createTextNode(item),
    // Update callback
    (node, item) => node.nodeValue = item + ' !!!'
)
```

## styles
```javascript
import styles from 'stage0/styles'

// Small CSS-in-JS utility for generating classNames and corresponding cssRules in document.head
const s = styles({
    base: {
        display: 'flex',
        // pseudo-classes and pseudo-selectors are supported
        '::before': {
            content: '>'
        }
    }
})
// s will have s.base === 'base-a'
// styles will generate uniq alphabet tokens and append it to the end of className
```
