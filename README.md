# @findify/react-bundle
React hook that allows you to integrate Findify Search/Autocomplete/Smart Collections/Recommendations with any React SPA App.
​
## Installation
​
#### Install dependency
```bash
yarn add @findify/react-bundle
```
​
#### Add script to `index.html`
```html
<script type="text/javascript" src="https://assets.findify.io/%REACT_APP_FINDIFY_STORE%.min.js"></script>
​
```
​
#### Add variable in `.env`
```
REACT_APP_FINDIFY_STORE="YOUR_STORE_NAME"
```
---
## Usage
```javascript
import React from 'react'
import useFindify from '@findify/react-bundle'
​
export default () => {
  const [container, isReady, hasError] = useFindify({ type: 'search' })
  return (
    <div ref={container} />
  )
}
```
​
The first element of an array is a `React.createRef()` that you should add as `ref` to the element where widget will be rendered.
​
The second element is a widget ready state: `true` or `false`. You can use this state to show and hide placeholders while findify is still rendering.
​
The third element of an array is an error state: `true` or `false`. It will return `true` if there are errors or if there are no items returned in response.
​
### Props
​
| Prop | Required | Default value | Description |
|---|---|---|---|
| `type<String>` | *yes* | `''`| Type of widget. It could be `search`, `recommendation`, `autocomplete`, `smart-collection`. |
| `history<History>` | *no* | `undefined`| Custom history instance. This prop is required if you want to have a history navigation. |
| `widgetKey<String>` | *no* | `random`| Widget unique Id. You can provide this parameter if you would like to use Findify API in the future. |
| `config<Object>` | *no* | | Custom widget configuration that will be merged with the default one. When it comes to creating Recommendation widgets, you would need to provide `slot` prop to the `config` object. |
| `options<Object>` | *no* | | Additional request options |
​
> For more information about Findify API and request options please visit [https://developers.findify.io/docs/merchant-js-api](https://developers.findify.io/docs/merchant-js-api)
​
​
---
## Simple Jetshop example
​
> To prevent unnecessary scroll add 'FindifyUpdate' to the `ignoreForRouteTypes` array in `/src/components/Shop.js`
```javascript
<ScrollRestorationHandler
  ignoreForRouteTypes={[
    'sortOrderChange',
    'filterChange',
    'paginationChange',
    'FindifyUpdate'
  ]}
/>
```
​
### Search
Go to `/components/SearchPage/SearchPage.js` 
```javascript
//...
import useFindify from '@findify/react-bundle';
import { useHistory } from 'react-router;
​
export const Container = styled(MaxWidth)`
  padding: 0 0.75rem;
`;
​
//...
const SearchPage = routeProps => {
  const track = useTracker();
  const history = useHistory();
  const { pathname, search } = routeProps.location;
  const [container, isReady] = useFindify({ type: 'search', history });
​
  useEffect(() => {
    track(trackPageEvent({ pathname: `${pathname}${search}` }));
  }, [track, pathname, search]);
​
  return (
    <Container>
      <div ref={container} />
    </Container>
  );
};
```
​
### Autocomplete
Go to `/components/Layout/Header/SearchBar.js`
```javascript
//...
import useFindify from '@findify/react-bundle';
import { useHistory } from 'react-router;
​
​
//...
const StyledSearchField = styled('div')`
  & {
    display: flex;
    height: 2rem;
    width: 100% !important;
    justify-content: center;
    input {
      border: 0;
      background: #f3f3f3;
      height: 100%;
      width: 100%;
      padding: 0 1rem;
    }
  }
`;
​
const SearchBar = ({ searchOpen, setSearchOpen }) => {
  const history = useHistory();
  const [container] = useFindify({ type: 'autocomplete', history });
  return (
    <PoseGroup flipMove={true}>
      {searchOpen && (
        <PosedSearchBarContainer key={'searchBarPosed'}>
          <InnerSearchBar>
            <StyledSearchField>
              <input ref={container} />
            </StyledSearchField>
          </InnerSearchBar>
        </PosedSearchBarContainer>
      )}
    </PoseGroup>
  )
};
```
​
### Recommendation
Go to `/components/StartPage/StatPage.js`
```javascript
//...
import useFindify from '@findify/react-bundle';
​
//...
const StartPage = ({ location: { pathname, key }, startPageId }) => {
  const [container, isReady, hasError] = useFindify({
    type: 'recommendation',
    config: {
      slot: 'home-findify-rec-2', // Slot is required for recommendation widgets
    }
  });
  return (
    <Fragment>
      <StartPageWrapper>
        <Container>
          <div ref={container} />
          { !isReady && !hasError && 'Widget loading...'}
          // ...
```
​
### Smart Collection
Go to `/components/StartPage/StatPage.js`
```javascript
//...
import useFindify from '@findify/react-bundle';
​
//...
​
const CategoryPage = props => {
  // ...
  const [container, isReady, hasError] = useFindify({ type: 'smart-collection' });
​
  if (!hasError) {
    return (
      <Container>
        <div ref={container} />
        { !isReady && 'Loading collection'}
      </Container>
    )
  }
  if (infinitePagination) {
    return <LoadableWindowedCategoryPage {...props} />;
  } else {
    return <LoadableStandardCategoryPage {...props} />;
  }
};
  //...
```
For Smart Collections, Findify is introducing fallback measurements by rendering the default collections, in case if the current Smart Collection is not setup in Findify. This is to prevent blank collection pages.

## Content in the Search Results
```javascript
//...
import useFindify from '@findify/react-bundle';
​
//...
​
const ContentPage = props => {
  // ...
  const [container, isReady, hasError] = useFindify({ type: 'content' });
​
  if (!hasError) {
    return (
      <Container>
        <div
          ref={container}
          data-type="shopify-collection_985"
         />
        { !isReady && 'Loading content'}
      </Container>
    )
  }
};
  //...
```
Unlike other widgets, content results widget get the type of content from the node where it is rendered.
You must provide `data-type="CONTENT_SOURCE"` to the element where is will be rendered (you can get the CONTENT_SOURCE parameter from the Merchant Dashboard).

## Analytics
To access Findify's analytics instance from anywhere in your app you can use the following example:
```javascript
import { waitForFindify } from '@findify/react-bundle';

async () => {
  const { analytics } = await waitForFindify();
  analytics.sendEvent('event', { ...options })
}
``` 

### Update cart event
Should be sent after product has been added to the cart and contain the whole cat content
```javascript
 const { analytics } = await waitForFindify();
 analytics.sendEvent('update-cart', {
    line_items: [ // Array of products
      {
        item_id: "PRODUCT_ID_1",
        quantity: 1,
        unit_price: 22.35,
        variant_item_id: "VARIANT_ID_1"
      }
    ]
 });
```
### Purchase event
Should be sent when user purchases products
```javascript
 const { analytics } = await waitForFindify();
 analytics.sendEvent('purchase', {
    currency: "EUR",
    line_items: [// Array of products
      {
        item_id: "PRODUCT_ID_1",
        quantity: 1,
        unit_price: 288.28,
        variant_item_id: "VARIANT_ID_1"
      },
    ],
    order_id: "ORDER_ID",
    revenue: 288.28
 });
```
### View page event
Should be sent every time user lands on the product page
```javascript
 const { analytics } = await waitForFindify();
 analytics.sendEvent('view-page', {
  item_id: "PRODUCT_ID",
  variant_item_id: "PRODUCT_VARIANT_ID"
 })
```
### Product click event
In case you are using your own History instance, you should update `components/Cards/Product/view.tsx` and change `onClick` event for the product card (this is done via DevTools Extension as a customization to our platfom):
```javascript
const ProductCardView ({ item }) => {
  const onClick = useCallback((e) => {
    e.preventDefault();
    // Calling this method will ensure that all analytics events will be sent to Findify properly
    item.sendAnalytics();
    findify.utils.history.push(item.get('product_url'))
  }, []);

  return (
    <a onClick={onClick}>
    ...
```
