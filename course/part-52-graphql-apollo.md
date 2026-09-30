# Part 52: GraphQL กับ Apollo Client

> **ระดับ:** มืออาชีพ / Professional  
> **Steps:** 1681-1720  
> **เวลาเรียน:** ~5 ชั่วโมง

---

## 📚 Table of Contents

1. [GraphQL vs REST](#graphql-vs-rest)
2. [Apollo Client Setup](#apollo-client-setup)
3. [Queries](#queries)
4. [Mutations](#mutations)
5. [Subscriptions](#subscriptions)
6. [useQuery, useMutation](#usequery-usemutation)
7. [Apollo Cache](#apollo-cache)
8. [Code Generation](#code-generation)
9. [ตัวอย่าง Blog App GraphQL](#blog-app-graphql)
10. [Quiz](#quiz)

---

## Step 1681: GraphQL vs REST {#graphql-vs-rest}

```
REST:
├── หลาย endpoints
├── Over-fetching (ได้ข้อมูลมากกว่าต้องการ)
├── Under-fetching (ต้อง request หลายครั้ง)
└── เปลี่ยน schema ยาก

GET /users/1           → { id, name, email, address, phone, ... }
GET /users/1/posts     → [{ id, title, body, ... }]
GET /posts/5/comments  → [{ id, text, author, ... }]

GraphQL:
├── Single endpoint (/graphql)
├── ขอเฉพาะข้อมูลที่ต้องการ
├── ดึงข้อมูลหลาย resource ใน request เดียว
└── Type-safe schema

query {
  user(id: "1") {
    name
    email
    posts(first: 5) {
      title
      comments {
        text
        author { name }
      }
    }
  }
}
```

### GraphQL Schema Basics

```graphql
# schema.graphql

# Scalar Types
scalar DateTime
scalar JSON
scalar Upload

# Enums
enum UserRole {
  ADMIN
  USER
  MODERATOR
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

# Object Types
type User {
  id: ID!
  name: String!
  email: String!
  avatar: String
  role: UserRole!
  posts: [Post!]!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  slug: String!
  excerpt: String
  content: String!
  status: PostStatus!
  author: User!
  tags: [String!]!
  coverImage: String
  publishedAt: DateTime
  createdAt: DateTime!
  updatedAt: DateTime!
  _count: PostCount!
}

type PostCount {
  comments: Int!
  likes: Int!
}

type Comment {
  id: ID!
  text: String!
  author: User!
  post: Post!
  createdAt: DateTime!
}

# Input Types
input CreatePostInput {
  title: String!
  content: String!
  excerpt: String
  status: PostStatus = DRAFT
  tags: [String!]
  coverImage: String
}

input UpdatePostInput {
  title: String
  content: String
  excerpt: String
  status: PostStatus
  tags: [String!]
  coverImage: String
}

input PostFilters {
  status: PostStatus
  authorId: ID
  tags: [String!]
  search: String
}

input Pagination {
  skip: Int = 0
  take: Int = 10
}

# Query Type
type Query {
  # Users
  me: User
  user(id: ID!): User
  users(filters: UserFilters, pagination: Pagination): UserConnection!
  
  # Posts
  post(id: ID, slug: String): Post
  posts(filters: PostFilters, pagination: Pagination, orderBy: PostOrderBy): PostConnection!
  featuredPosts: [Post!]!
  
  # Comments
  comments(postId: ID!): [Comment!]!
}

# Mutation Type
type Mutation {
  # Auth
  login(email: String!, password: String!): AuthPayload!
  register(input: RegisterInput!): AuthPayload!
  logout: Boolean!
  
  # Posts
  createPost(input: CreatePostInput!): Post!
  updatePost(id: ID!, input: UpdatePostInput!): Post!
  deletePost(id: ID!): Boolean!
  publishPost(id: ID!): Post!
  
  # Comments
  addComment(postId: ID!, text: String!): Comment!
  deleteComment(id: ID!): Boolean!
  
  # Likes
  toggleLike(postId: ID!): Post!
}

# Subscription Type
type Subscription {
  newComment(postId: ID!): Comment!
  postUpdated(id: ID!): Post!
  newNotification: Notification!
}

# Connection Pattern (Pagination)
type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type PostEdge {
  node: Post!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

---

## Step 1682-1688: Apollo Client Setup {#apollo-client-setup}

### Installation

```bash
npm install @apollo/client graphql
npm install -D @graphql-codegen/cli @graphql-codegen/typescript @graphql-codegen/typescript-operations @graphql-codegen/typescript-react-apollo
```

### Apollo Client Configuration

```typescript
// lib/apolloClient.ts
import { ApolloClient, InMemoryCache, ApolloLink, from, split } from '@apollo/client'
import { setContext } from '@apollo/client/link/context'
import { onError } from '@apollo/client/link/error'
import { GraphQLWsLink } from '@apollo/client/link/subscriptions'
import { createClient } from 'graphql-ws'
import { getMainDefinition } from '@apollo/client/utilities'
import createUploadLink from 'apollo-upload-client/createUploadLink.mjs'
import { relayStylePagination } from '@apollo/client/utilities'

const httpLink = createUploadLink({
  uri: process.env.NEXT_PUBLIC_GRAPHQL_URL || 'http://localhost:4000/graphql',
  credentials: 'include',
  headers: {
    'Apollo-Require-Preflight': 'true',
  },
})

const authLink = setContext((_, { headers }) => {
  const token = typeof window !== 'undefined' 
    ? localStorage.getItem('token') 
    : null
  
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
    },
  }
})

const wsLink = typeof window !== 'undefined'
  ? new GraphQLWsLink(
      createClient({
        url: process.env.NEXT_PUBLIC_GRAPHQL_WS_URL || 'ws://localhost:4000/graphql',
        connectionParams: () => {
          const token = localStorage.getItem('token')
          return { authorization: token ? `Bearer ${token}` : '' }
        },
      })
    )
  : null

const errorLink = onError(({ graphQLErrors, networkError, operation, forward }) => {
  if (graphQLErrors) {
    for (const err of graphQLErrors) {
      switch (err.extensions?.code) {
        case 'UNAUTHENTICATED':
          // Refresh token หรือ redirect ไป login
          localStorage.removeItem('token')
          window.location.href = '/login'
          break
        case 'FORBIDDEN':
          console.error('Permission denied:', err.message)
          break
        default:
          console.error('[GraphQL error]:', err)
      }
    }
  }
  
  if (networkError) {
    console.error('[Network error]:', networkError)
  }
})

const splitLink = wsLink
  ? split(
      ({ query }) => {
        const definition = getMainDefinition(query)
        return (
          definition.kind === 'OperationDefinition' &&
          definition.operation === 'subscription'
        )
      },
      wsLink,
      from([errorLink, authLink, httpLink])
    )
  : from([errorLink, authLink, httpLink])

export const createApolloClient = () =>
  new ApolloClient({
    ssrMode: typeof window === 'undefined',
    link: splitLink,
    cache: new InMemoryCache({
      typePolicies: {
        Query: {
          fields: {
            posts: relayStylePagination(['filters', 'orderBy']),
            users: relayStylePagination(),
          },
        },
        Post: {
          keyFields: ['id'],
          fields: {
            comments: {
              merge: false,
            },
          },
        },
      },
    }),
    defaultOptions: {
      watchQuery: {
        fetchPolicy: 'cache-and-network',
        errorPolicy: 'all',
      },
      query: {
        fetchPolicy: 'network-only',
        errorPolicy: 'all',
      },
      mutate: {
        errorPolicy: 'all',
      },
    },
  })
```

### Apollo Provider

```typescript
// providers/ApolloProvider.tsx
'use client'

import { ApolloProvider as BaseApolloProvider } from '@apollo/client'
import { useRef } from 'react'
import { createApolloClient } from '@/lib/apolloClient'

export function ApolloProvider({ children }: { children: React.ReactNode }) {
  const clientRef = useRef(createApolloClient())
  
  return (
    <BaseApolloProvider client={clientRef.current}>
      {children}
    </BaseApolloProvider>
  )
}

// app/layout.tsx
import { ApolloProvider } from '@/providers/ApolloProvider'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <ApolloProvider>
          {children}
        </ApolloProvider>
      </body>
    </html>
  )
}
```

---

## Step 1689-1698: Queries, Mutations, Subscriptions {#queries}

### GraphQL Queries

```typescript
// graphql/queries/posts.ts
import { gql } from '@apollo/client'

export const POST_FRAGMENT = gql`
  fragment PostFields on Post {
    id
    title
    slug
    excerpt
    status
    coverImage
    publishedAt
    createdAt
    _count {
      comments
      likes
    }
    author {
      id
      name
      avatar
    }
    tags
  }
`

export const GET_POSTS = gql`
  ${POST_FRAGMENT}
  query GetPosts(
    $filters: PostFilters
    $pagination: Pagination
    $orderBy: PostOrderBy
  ) {
    posts(filters: $filters, pagination: $pagination, orderBy: $orderBy) {
      edges {
        node {
          ...PostFields
        }
        cursor
      }
      pageInfo {
        hasNextPage
        endCursor
      }
      totalCount
    }
  }
`

export const GET_POST = gql`
  query GetPost($id: ID, $slug: String) {
    post(id: $id, slug: $slug) {
      id
      title
      slug
      content
      excerpt
      status
      coverImage
      publishedAt
      createdAt
      updatedAt
      tags
      author {
        id
        name
        email
        avatar
      }
      _count {
        comments
        likes
      }
    }
  }
`

export const GET_FEATURED_POSTS = gql`
  ${POST_FRAGMENT}
  query GetFeaturedPosts {
    featuredPosts {
      ...PostFields
    }
  }
`
```

### GraphQL Mutations

```typescript
// graphql/mutations/posts.ts
import { gql } from '@apollo/client'

export const CREATE_POST = gql`
  mutation CreatePost($input: CreatePostInput!) {
    createPost(input: $input) {
      id
      title
      slug
      status
      createdAt
    }
  }
`

export const UPDATE_POST = gql`
  mutation UpdatePost($id: ID!, $input: UpdatePostInput!) {
    updatePost(id: $id, input: $input) {
      id
      title
      slug
      status
      updatedAt
    }
  }
`

export const DELETE_POST = gql`
  mutation DeletePost($id: ID!) {
    deletePost(id: $id)
  }
`

export const PUBLISH_POST = gql`
  mutation PublishPost($id: ID!) {
    publishPost(id: $id) {
      id
      status
      publishedAt
    }
  }
`

export const ADD_COMMENT = gql`
  mutation AddComment($postId: ID!, $text: String!) {
    addComment(postId: $postId, text: $text) {
      id
      text
      createdAt
      author {
        id
        name
        avatar
      }
    }
  }
`

export const TOGGLE_LIKE = gql`
  mutation ToggleLike($postId: ID!) {
    toggleLike(postId: $postId) {
      id
      _count {
        likes
      }
    }
  }
`
```

### GraphQL Subscriptions

```typescript
// graphql/subscriptions/posts.ts
import { gql } from '@apollo/client'

export const NEW_COMMENT_SUBSCRIPTION = gql`
  subscription NewComment($postId: ID!) {
    newComment(postId: $postId) {
      id
      text
      createdAt
      author {
        id
        name
        avatar
      }
    }
  }
`

export const POST_UPDATED_SUBSCRIPTION = gql`
  subscription PostUpdated($id: ID!) {
    postUpdated(id: $id) {
      id
      title
      content
      status
      updatedAt
    }
  }
`
```

---

## Step 1699-1707: useQuery, useMutation {#usequery-usemutation}

### useQuery

```typescript
// components/PostList.tsx
'use client'

import { useQuery } from '@apollo/client'
import { GET_POSTS } from '@/graphql/queries/posts'
import type { GetPostsQuery, GetPostsQueryVariables, PostStatus } from '@/generated/graphql'

export function PostList() {
  const { data, loading, error, fetchMore, refetch } = useQuery<
    GetPostsQuery,
    GetPostsQueryVariables
  >(GET_POSTS, {
    variables: {
      filters: { status: PostStatus.Published },
      pagination: { skip: 0, take: 10 },
    },
    notifyOnNetworkStatusChange: true,
  })
  
  if (loading && !data) return <PostSkeleton />
  if (error) return <ErrorMessage message={error.message} />
  
  const { edges, pageInfo, totalCount } = data!.posts
  
  return (
    <div>
      <p className="text-gray-500 mb-4">{totalCount} บทความ</p>
      
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {edges.map(({ node: post }) => (
          <PostCard key={post.id} post={post} />
        ))}
      </div>
      
      {pageInfo.hasNextPage && (
        <button
          onClick={() =>
            fetchMore({
              variables: {
                pagination: { skip: edges.length, take: 10 },
              },
            })
          }
          className="mt-8 w-full bg-gray-100 hover:bg-gray-200 py-3 rounded-lg"
        >
          {loading ? 'กำลังโหลด...' : 'โหลดเพิ่มเติม'}
        </button>
      )}
    </div>
  )
}
```

### useMutation

```typescript
// components/PostForm.tsx
'use client'

import { useMutation } from '@apollo/client'
import { CREATE_POST, GET_POSTS } from '@/graphql'
import type { CreatePostMutation, CreatePostMutationVariables } from '@/generated/graphql'
import { useRouter } from 'next/navigation'

export function PostForm() {
  const router = useRouter()
  const [title, setTitle] = useState('')
  const [content, setContent] = useState('')
  
  const [createPost, { loading, error }] = useMutation<
    CreatePostMutation,
    CreatePostMutationVariables
  >(CREATE_POST, {
    onCompleted: (data) => {
      router.push(`/blog/${data.createPost.slug}`)
    },
    onError: (error) => {
      console.error('Error creating post:', error)
    },
    // Update cache หลัง mutation
    update: (cache, { data }) => {
      if (!data?.createPost) return
      
      // อ่าน cache ปัจจุบัน
      const existing = cache.readQuery<any>({
        query: GET_POSTS,
        variables: { filters: {} },
      })
      
      if (existing) {
        // เขียน cache ใหม่
        cache.writeQuery({
          query: GET_POSTS,
          variables: { filters: {} },
          data: {
            posts: {
              ...existing.posts,
              edges: [
                { node: data.createPost, cursor: data.createPost.id },
                ...existing.posts.edges,
              ],
              totalCount: existing.posts.totalCount + 1,
            },
          },
        })
      }
    },
    // หรือ refetch queries
    // refetchQueries: [{ query: GET_POSTS, variables: { filters: {} } }],
    // awaitRefetchQueries: true,
  })
  
  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault()
    
    await createPost({
      variables: {
        input: { title, content },
      },
    })
  }
  
  return (
    <form onSubmit={handleSubmit}>
      {error && <p className="text-red-500">{error.message}</p>}
      
      <input
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="หัวข้อบทความ"
      />
      
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder="เนื้อหา"
      />
      
      <button type="submit" disabled={loading}>
        {loading ? 'กำลังบันทึก...' : 'สร้างบทความ'}
      </button>
    </form>
  )
}
```

### useSubscription

```typescript
// components/CommentList.tsx
'use client'

import { useQuery, useSubscription } from '@apollo/client'
import { GET_COMMENTS, NEW_COMMENT_SUBSCRIPTION } from '@/graphql'

export function CommentList({ postId }: { postId: string }) {
  const { data: queryData, loading } = useQuery(GET_COMMENTS, {
    variables: { postId },
  })
  
  const [comments, setComments] = useState(queryData?.comments || [])
  
  useEffect(() => {
    if (queryData?.comments) {
      setComments(queryData.comments)
    }
  }, [queryData])
  
  // Subscribe to new comments
  useSubscription(NEW_COMMENT_SUBSCRIPTION, {
    variables: { postId },
    onData: ({ data }) => {
      const newComment = data.data?.newComment
      if (newComment) {
        setComments((prev) => [...prev, newComment])
      }
    },
  })
  
  return (
    <div className="space-y-4">
      {loading ? (
        <p>กำลังโหลด...</p>
      ) : (
        comments.map((comment) => (
          <div key={comment.id} className="p-4 bg-gray-50 rounded-lg">
            <div className="flex items-center gap-2 mb-2">
              <img src={comment.author.avatar} className="w-8 h-8 rounded-full" />
              <span className="font-medium">{comment.author.name}</span>
              <span className="text-sm text-gray-500">
                {new Date(comment.createdAt).toLocaleDateString('th-TH')}
              </span>
            </div>
            <p>{comment.text}</p>
          </div>
        ))
      )}
    </div>
  )
}
```

---

## Step 1708-1713: Apollo Cache {#apollo-cache}

### Cache Policy

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        // Cursor-based pagination
        posts: relayStylePagination(['filters', 'orderBy']),
        
        // Offset pagination
        users: {
          keyArgs: ['filters'],
          merge(existing = { edges: [] }, incoming) {
            return {
              ...incoming,
              edges: [...existing.edges, ...incoming.edges],
            }
          },
        },
        
        // Single item
        post: {
          read(_, { args, toReference }) {
            return toReference({
              __typename: 'Post',
              id: args?.id,
            })
          },
        },
      },
    },
    
    Post: {
      keyFields: ['id'],
      fields: {
        // Transform field
        publishedAt: {
          read(value) {
            return value ? new Date(value) : null
          },
        },
        
        // Local-only field (ไม่ได้มาจาก server)
        isLiked: {
          read(value = false) {
            return value
          },
        },
      },
    },
    
    User: {
      keyFields: ['id'],
    },
  },
})
```

### Cache Manipulation

```typescript
// อ่านจาก cache โดยตรง
function PostCard({ postId }: { postId: string }) {
  const { cache } = useApolloClient()
  
  const post = cache.readFragment({
    id: `Post:${postId}`,
    fragment: gql`
      fragment PostPreview on Post {
        id
        title
        status
      }
    `,
  })
  
  return <div>{post?.title}</div>
}

// อัพเดท cache หลัง delete
const [deletePost] = useMutation(DELETE_POST, {
  update: (cache, { data }) => {
    if (!data?.deletePost) return
    
    // Evict จาก cache
    cache.evict({ id: `Post:${postId}` })
    cache.gc() // Garbage collect
  },
})

// Fragment Update
const [toggleLike] = useMutation(TOGGLE_LIKE, {
  optimisticResponse: {
    toggleLike: {
      __typename: 'Post',
      id: postId,
      _count: {
        __typename: 'PostCount',
        likes: isLiked ? currentLikes - 1 : currentLikes + 1,
      },
    },
  },
})
```

---

## Step 1714-1716: Code Generation {#code-generation}

### Setup GraphQL Code Generator

```yaml
# codegen.yml
overwrite: true
schema: "http://localhost:4000/graphql"
documents: "src/**/*.{ts,tsx,graphql}"
generates:
  src/generated/graphql.ts:
    plugins:
      - "typescript"
      - "typescript-operations"
      - "typescript-react-apollo"
    config:
      withHooks: true
      withHOC: false
      withComponent: false
      strictScalars: true
      scalars:
        DateTime: string
        JSON: Record<string, any>
        Upload: File
```

```bash
# Generate types
npx graphql-codegen --config codegen.yml

# Watch mode
npx graphql-codegen --config codegen.yml --watch
```

---

## Step 1717-1720: ตัวอย่าง Blog App GraphQL {#blog-app-graphql}

### Blog Post Detail Page

```typescript
// app/blog/[slug]/page.tsx
import { createApolloClient } from '@/lib/apolloClient'
import { GET_POST } from '@/graphql/queries/posts'
import type { GetPostQuery } from '@/generated/graphql'
import { PostContent } from '@/components/PostContent'
import { CommentSection } from '@/components/CommentSection'

export async function generateMetadata({ params }: { params: { slug: string } }) {
  const client = createApolloClient()
  const { data } = await client.query<GetPostQuery>({
    query: GET_POST,
    variables: { slug: params.slug },
  })
  
  return {
    title: data?.post?.title,
    description: data?.post?.excerpt,
    openGraph: {
      images: [data?.post?.coverImage],
    },
  }
}

export default async function BlogPostPage({
  params,
}: {
  params: { slug: string }
}) {
  const client = createApolloClient()
  const { data } = await client.query<GetPostQuery>({
    query: GET_POST,
    variables: { slug: params.slug },
  })
  
  if (!data?.post) {
    notFound()
  }
  
  const post = data.post
  
  return (
    <article className="max-w-3xl mx-auto py-12 px-4">
      {post.coverImage && (
        <img
          src={post.coverImage}
          alt={post.title}
          className="w-full h-64 object-cover rounded-xl mb-8"
        />
      )}
      
      <header className="mb-8">
        <div className="flex gap-2 mb-4">
          {post.tags.map((tag) => (
            <span key={tag} className="px-3 py-1 bg-blue-100 text-blue-600 rounded-full text-sm">
              {tag}
            </span>
          ))}
        </div>
        
        <h1 className="text-3xl md:text-4xl font-bold mb-4">{post.title}</h1>
        
        <div className="flex items-center gap-4 text-gray-500 text-sm">
          <div className="flex items-center gap-2">
            <img src={post.author.avatar!} className="w-8 h-8 rounded-full" />
            <span>{post.author.name}</span>
          </div>
          <span>•</span>
          <span>
            {new Date(post.publishedAt!).toLocaleDateString('th-TH', {
              year: 'numeric',
              month: 'long',
              day: 'numeric',
            })}
          </span>
          <span>•</span>
          <span>{post._count.comments} ความคิดเห็น</span>
        </div>
      </header>
      
      <PostContent content={post.content} />
      
      <CommentSection postId={post.id} />
    </article>
  )
}
```

---

## 🧪 Quiz - Part 52

**ข้อ 1:** GraphQL แก้ปัญหา Over-fetching ใน REST API อย่างไร?
- A) ใช้ cache ดีกว่า
- B) Client กำหนดได้ว่าต้องการ fields ใด
- C) ใช้ compression
- D) ใช้ HTTP/2

**ข้อ 2:** `relayStylePagination` ใน Apollo cache ใช้สำหรับ?
- A) Infinite scroll pagination แบบ cursor-based
- B) Offset pagination
- C) Page-based pagination
- D) Real-time updates

**ข้อ 3:** `optimisticResponse` ใน useMutation คืออะไร?
- A) Retry อัตโนมัติถ้า mutation ล้มเหลว
- B) อัพเดท UI ทันทีก่อนที่ server จะตอบ
- C) Cache mutation ไว้ก่อน
- D) Validate input ก่อนส่ง

**ข้อ 4:** GraphQL Subscription แตกต่างจาก Query อย่างไร?
- A) Subscription ใช้ HTTP POST
- B) Subscription รับข้อมูล real-time ผ่าน WebSocket
- C) Subscription เร็วกว่า Query
- D) ไม่มีความแตกต่าง

**เฉลย:** 1-B, 2-A, 3-B, 4-B

---

> **➡️ Next:** [Part 53: Next.js Edge Runtime](./part-53-nextjs-edge-runtime.md)
