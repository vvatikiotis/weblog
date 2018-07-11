---
Author:
  - testtag
---

# Compound components in React

- React doesn't add/remove event listeners, it just delegates events to a parent node event listener. Rather than adding an event listener to a DOM element, React provides a listener when a child element mounts or unmounts.
  - Found this snippet from a [Dan's answer](https://github.com/facebook/react/issues/7094):
    > Event delegation: React doesn't actually attach event handlers to the nodes themselves. When React starts up, it starts listening for all events at the top level using a single event listener. When a component is mounted or unmounted, the event handlers are simply added or removed from an internal mapping. When an event occurs, React knows how to dispatch it using this mapping. When there are no event handlers left in the mapping, React's event handlers are simple no-ops. To learn more about why this is fast, see David Walsh's excellent blog post.

An example of a compound component:

```jsx
<Tabs>
  <Tablist>
    <Tab>
      <Payment />
    </Tab>
    <Tab>
      <Order />
    </Tab>
    <Tab>
      <DebitCard />
    </Tab> <Tab>
      <CreditCard />
    </Tab>
  </Tablist>
  <TabPanels>
    <TabPanel>
      <PaymentPanel />
    </TabPanel>
    <TabPanel>
      <OrderPanel />
    </TabPanel>
    <TabPanel>
      <DebitPanel />
    </TabPanel>
    <TabPanel>
      <CreditPanel />
    </TabPanel>
  </TabPanels>
</Tabs>
```

- Implicit state: `cloneElement` and pass the implicit state as a prop. We can also pass handlers. So state and handlers live in the top level component `Tabs`

**Using compound components enables up to wrap our new component into our old component**. Our top level, old API has a very short footprint i.e. very few or a single prop, e.g. just pass an array of data or an object. But then, the API _decompose_ to smaller, composable, independent APIs. So build components using small decomposable pieces, which you can add, move or remove at will. That is compound components.

**Start with state! Then the handlers.**

**If a component doesn't own the state then there has to be a handler that change a state somewhere else. The handler has to come from outside the component, from `props`**
