---
title: How to get type from GraphQL response
tags:
  - GraphQL
  - TypeScript
creationDate: 1622352115
---

# How to get type from GraphQL response

This week I was working on REST to GraphQL migration. I went all the way and used  `graphql-codegen` to simplify (hah) my life by generating TypeScript types from `.gql` files. Also, I used [aliases](https://graphql.org/learn/queries/#aliases), since our backend contract had slightly changed but I don't want to write mapper functions in my code: before, some of the fields had and `item` prefix, now it does not. So, you have some query like tiis:

```graphql
query GetCollection($collectionId: String) {
  items(
    collectionId: $collectionId
    ) {
    items {
      itemId: id,
      itemTitle: title,
      price,
    }
  }
}
```

And you have some generated typescript code (only part of it presented):

```ts
export type Maybe<T> = T | null;

export type Item = {
  __typename?: 'Item';
  id: string;
  price: number;
  title: string;
  isApproved: boolean;
  isAvaliable: boolean;
};

export type GetCollectionQuery = (
  { __typename?: 'RootQueryType' }
  & { items?: Maybe<(
    { __typename?: 'GetCollectionPayload' }
    & { items: Array<(
      { __typename?: 'Item' }
      & Pick<Item, 'price'>
      & { itemId: Item['id'], itemTitle: Item['title'] }
    )> }
  )> }
);
```

Now, you can see that new `Item` type was generated for you, with all possible fields and new naming convention. But in the `GetCollectionQuery` I don't see an explicit `GetCollectionQuery` type, I see some scary `Maybe/Pick/['id']` combination. But you want to get the actual `Item` type which is returned by your query. So, I have decided to just copy-paste it from generated type:

```ts
import { Item } from 'generated.js'

export type ItemFromQuery = {
  __typename: 'Item'
  & Pick<Item, 'price'>
  & { itemId: Item['id'], itemTitle: Item['title'] }
}
```

This way works. But now, if I am to change the query, I will need to also change this code. So, let's try to get this type from generated code. There should be a way to do something like `getTypeOf GetCollectionQuery.items.items[0]` in TypeScript, right? With [Indexed Access Types](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html) I can do exactly that. With them, I could get a type of a specific property from other type:

```ts
type OuterItems = GetCollectionQuery["items"]
```

Looks fine. Let's try to get __inner__ items array, using the same way:

```ts
type InnerItems = OuterItems["items"]
```

Well, it doesn't seem to work:
```
Property 'items' does not exist on type '({ __typename?: "ItemsPayload" | undefined; } & { items: ({ __typename?: "Item" | undefined; } & Pick<Item, "price"> & { itemId: string; itemTitle: string; })[]; }) | null | undefined'.ts(2339)
```

Apparently, `Maybe` and `?` don't let me do it: items array could be not an array, but rather `null` or `undefined`. I need some way to __un-maybe__ it. [NonNullable](https://www.typescriptlang.org/docs/handbook/utility-types.html#nonnullabletype) utility type could help me with that:

```ts
type InnerItems = NonNullable<OuterItems>["items"]
```

Now it seems to work. But how can I get a type of items of this array? [Indexed Access Types](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html) will help me again:
```ts
type ItemFromQuery = InnerItems[number]
```
`SomeArray[number]` is an __Indexed Access Type__ too. It helps me to get a type of array elements. I don't need need a `typeof` keyword (unlike the example from the TypeScript Handbook), since we work with types.

Combining all the parts we get:
```ts
type ItemFromQuery =  NonNullable<GetCollectionQuery["items"]>["items"][number]
```

To sum up, today I learned:
- [Indexed Access Types](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html)
- [NonNullable](https://www.typescriptlang.org/docs/handbook/utility-types.html#nonnullabletype) utility type