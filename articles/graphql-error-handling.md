---
title: "TODO"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["graphql"]
published: true
published_at: 2024-12-05 09:00
publication_name: "moneyforward"
---
:::message
この記事は、[Money Forward Engineers Advent Calendar 2024](https://adventar.org/calendars/9988)の5日目の記事です。
:::

私のプロジェクトで採用しているGraphQLスキーマ設計について紹介します。

```graphql
type Mutation {
  completeTask(input: CompleteTaskInput!): CompleteTaskResult!
}

input CompleteTaskInput {
  id: ID!
}

interface UserError {
  message: String!
}

type UnauthorizedError implements UserError {
  message: String!
}
type NotFoundError implements UserError {
  message: String!
}
type OptimisticLockError implements UserError {
  message: String!
}

type Error implements UserError {
  message: String!
}

union CompleteTaskResult = Task | CompleteTaskErrors
union CompleteTaskError =
  | UnauthorizedError
  | NotFoundError
  | OptimisticLockError

type CompleteTaskErrors {
  errors: [CompleteTaskError!]!
}

type Task {
  id: ID!
  status: TaskStatus!
}

```
