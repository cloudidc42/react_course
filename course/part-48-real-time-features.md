# Part 48: Real-time Features ใน React และ Next.js

> **ระดับ:** มืออาชีพ / Professional  
> **Steps:** 1511-1550  
> **เวลาเรียน:** ~5 ชั่วโมง

---

## 📚 Table of Contents

1. [WebSocket คืออะไร](#websocket-คืออะไร)
2. [Socket.io กับ React](#socketio-กับ-react)
3. [Server-Sent Events (SSE)](#server-sent-events)
4. [Next.js Route Handler + SSE](#nextjs-route-handler-sse)
5. [Real-time Chat App](#real-time-chat-app)
6. [Live Notifications](#live-notifications)
7. [Collaborative Features](#collaborative-features)
8. [Quiz](#quiz)

---

## Step 1511: WebSocket คืออะไร {#websocket-คืออะไร}

WebSocket เป็น Protocol ที่ช่วยให้ Browser และ Server สื่อสารกันแบบ **สองทาง (Full-duplex)** ผ่าน Connection เดียว

### HTTP vs WebSocket vs SSE

```
HTTP (Request-Response):
Browser ──── Request ────► Server
Browser ◄─── Response ──── Server
(ต้องส่ง Request ใหม่ทุกครั้งที่ต้องการข้อมูล)

WebSocket (Full-duplex):
Browser ◄──── Connection ────► Server
Browser ──── Message ────►
Browser ◄─── Message ────
(สื่อสารได้ทั้งสองทางตลอดเวลา)

SSE (Server-Sent Events):
Browser ──── Request ────► Server
Browser ◄─── Stream ──── Server
Browser ◄─── Stream ──── Server
Browser ◄─── Stream ──── Server
(Server ส่งข้อมูลไปที่ Browser ได้ตลอด แต่ Browser ส่งกลับไม่ได้)
```

### เมื่อไรใช้อะไร

```
WebSocket:
✅ Chat application
✅ Online gaming
✅ Collaborative editing
✅ Real-time trading
✅ ต้องการสื่อสาร 2 ทาง

SSE:
✅ Live feeds
✅ Notifications
✅ Real-time dashboards
✅ Stock prices
✅ ข้อมูลไหลทาง Server → Client เท่านั้น
✅ ใช้ HTTP ปกติ (ง่ายกว่า)

Polling:
✅ ข้อมูลไม่ต้อง real-time มาก
✅ Infrastructure เรียบง่าย
✅ ข้อมูลเปลี่ยนแปลงไม่บ่อย
```

---

## Step 1512-1520: Socket.io กับ React {#socketio-กับ-react}

### Installation

```bash
# Server dependencies
npm install socket.io express

# Client dependencies
npm install socket.io-client
```

### Setup Socket.io Server

```typescript
// server/index.ts
import express from 'express'
import { createServer } from 'http'
import { Server } from 'socket.io'
import cors from 'cors'

const app = express()
const httpServer = createServer(app)

const io = new Server(httpServer, {
  cors: {
    origin: process.env.CLIENT_URL || 'http://localhost:3000',
    methods: ['GET', 'POST'],
    credentials: true,
  },
})

// Middleware Authentication
io.use((socket, next) => {
  const token = socket.handshake.auth.token
  
  if (!token) {
    return next(new Error('Authentication required'))
  }
  
  try {
    const user = verifyToken(token)
    socket.data.user = user
    next()
  } catch (error) {
    next(new Error('Invalid token'))
  }
})

// Event Handlers
io.on('connection', (socket) => {
  const user = socket.data.user
  console.log(`User connected: ${user.name} (${socket.id})`)
  
  // Join user to their room
  socket.join(`user:${user.id}`)
  
  // Notify online status
  socket.broadcast.emit('user:online', {
    userId: user.id,
    name: user.name,
  })
  
  // Handle join room
  socket.on('room:join', (roomId: string) => {
    socket.join(roomId)
    io.to(roomId).emit('room:user-joined', {
      userId: user.id,
      name: user.name,
    })
  })
  
  // Handle leave room
  socket.on('room:leave', (roomId: string) => {
    socket.leave(roomId)
    io.to(roomId).emit('room:user-left', {
      userId: user.id,
      name: user.name,
    })
  })
  
  // Handle chat message
  socket.on('message:send', async (data: {
    roomId: string
    content: string
  }) => {
    const message = await saveMessage({
      userId: user.id,
      roomId: data.roomId,
      content: data.content,
    })
    
    // Broadcast ไปทุกคนในห้อง
    io.to(data.roomId).emit('message:received', {
      id: message.id,
      content: message.content,
      sender: {
        id: user.id,
        name: user.name,
        avatar: user.avatar,
      },
      timestamp: message.createdAt,
    })
  })
  
  // Handle typing indicator
  socket.on('typing:start', (roomId: string) => {
    socket.to(roomId).emit('typing:user', {
      userId: user.id,
      name: user.name,
      isTyping: true,
    })
  })
  
  socket.on('typing:stop', (roomId: string) => {
    socket.to(roomId).emit('typing:user', {
      userId: user.id,
      name: user.name,
      isTyping: false,
    })
  })
  
  // Handle disconnect
  socket.on('disconnect', () => {
    console.log(`User disconnected: ${user.name}`)
    socket.broadcast.emit('user:offline', {
      userId: user.id,
    })
  })
})

httpServer.listen(3001, () => {
  console.log('Socket.io server running on port 3001')
})
```

### Socket.io Context (React)

```typescript
// contexts/SocketContext.tsx
'use client'

import { createContext, useContext, useEffect, useState } from 'react'
import { io, Socket } from 'socket.io-client'
import { useSession } from 'next-auth/react'

interface SocketContextType {
  socket: Socket | null
  isConnected: boolean
}

const SocketContext = createContext<SocketContextType>({
  socket: null,
  isConnected: false,
})

export function SocketProvider({ children }: { children: React.ReactNode }) {
  const [socket, setSocket] = useState<Socket | null>(null)
  const [isConnected, setIsConnected] = useState(false)
  const { data: session } = useSession()
  
  useEffect(() => {
    if (!session?.user) return
    
    const newSocket = io(process.env.NEXT_PUBLIC_SOCKET_URL!, {
      auth: {
        token: session.accessToken,
      },
      reconnection: true,
      reconnectionAttempts: 5,
      reconnectionDelay: 1000,
    })
    
    newSocket.on('connect', () => {
      console.log('Connected to Socket.io')
      setIsConnected(true)
    })
    
    newSocket.on('disconnect', () => {
      console.log('Disconnected from Socket.io')
      setIsConnected(false)
    })
    
    newSocket.on('connect_error', (error) => {
      console.error('Connection error:', error)
      setIsConnected(false)
    })
    
    setSocket(newSocket)
    
    return () => {
      newSocket.close()
    }
  }, [session])
  
  return (
    <SocketContext.Provider value={{ socket, isConnected }}>
      {children}
    </SocketContext.Provider>
  )
}

export const useSocket = () => useContext(SocketContext)
```

### Custom Hooks สำหรับ Socket.io

```typescript
// hooks/useChat.ts
import { useEffect, useState, useCallback, useRef } from 'react'
import { useSocket } from '@/contexts/SocketContext'
import { useSession } from 'next-auth/react'

interface Message {
  id: string
  content: string
  sender: {
    id: string
    name: string
    avatar?: string
  }
  timestamp: string
}

interface TypingUser {
  userId: string
  name: string
}

export function useChat(roomId: string) {
  const { socket } = useSocket()
  const { data: session } = useSession()
  const [messages, setMessages] = useState<Message[]>([])
  const [typingUsers, setTypingUsers] = useState<TypingUser[]>([])
  const [isLoading, setIsLoading] = useState(true)
  const typingTimeoutRef = useRef<NodeJS.Timeout>()
  
  // โหลด messages เก่า
  useEffect(() => {
    fetch(`/api/rooms/${roomId}/messages`)
      .then((res) => res.json())
      .then((data) => {
        setMessages(data.messages)
        setIsLoading(false)
      })
  }, [roomId])
  
  // Subscribe to socket events
  useEffect(() => {
    if (!socket) return
    
    // Join room
    socket.emit('room:join', roomId)
    
    // Listen for new messages
    const handleMessage = (message: Message) => {
      setMessages((prev) => [...prev, message])
    }
    socket.on('message:received', handleMessage)
    
    // Listen for typing
    const handleTyping = ({ userId, name, isTyping }: any) => {
      if (userId === session?.user?.id) return
      
      setTypingUsers((prev) => {
        if (isTyping) {
          if (prev.find((u) => u.userId === userId)) return prev
          return [...prev, { userId, name }]
        } else {
          return prev.filter((u) => u.userId !== userId)
        }
      })
    }
    socket.on('typing:user', handleTyping)
    
    return () => {
      socket.emit('room:leave', roomId)
      socket.off('message:received', handleMessage)
      socket.off('typing:user', handleTyping)
    }
  }, [socket, roomId, session])
  
  // Send message
  const sendMessage = useCallback(
    (content: string) => {
      if (!socket || !content.trim()) return
      
      socket.emit('message:send', { roomId, content })
      
      // Stop typing indicator
      socket.emit('typing:stop', roomId)
    },
    [socket, roomId]
  )
  
  // Typing indicator
  const startTyping = useCallback(() => {
    if (!socket) return
    
    socket.emit('typing:start', roomId)
    
    // Auto stop after 3 seconds
    clearTimeout(typingTimeoutRef.current)
    typingTimeoutRef.current = setTimeout(() => {
      socket.emit('typing:stop', roomId)
    }, 3000)
  }, [socket, roomId])
  
  return {
    messages,
    typingUsers,
    isLoading,
    sendMessage,
    startTyping,
  }
}
```

---

## Step 1521-1530: Server-Sent Events (SSE) {#server-sent-events}

SSE เป็นวิธีที่ง่ายกว่า WebSocket สำหรับการส่งข้อมูลจาก Server ไปยัง Client

### SSE Format

```
data: Hello World\n\n

data: {"type":"message","content":"Hello"}\n\n

event: update\n
data: {"status":"complete"}\n\n

id: 123\n
event: news\n
data: New article published\n\n

: comment line (ignored by client)\n\n

retry: 3000\n
data: reconnect in 3 seconds\n\n
```

### SSE Server ด้วย Node.js

```typescript
// server/sse.ts (Express)
import express from 'express'
const router = express.Router()

// Store ของ SSE connections
const clients = new Map<string, express.Response>()

router.get('/events', (req, res) => {
  const userId = req.query.userId as string
  
  // Set headers สำหรับ SSE
  res.setHeader('Content-Type', 'text/event-stream')
  res.setHeader('Cache-Control', 'no-cache')
  res.setHeader('Connection', 'keep-alive')
  res.setHeader('X-Accel-Buffering', 'no') // สำหรับ Nginx
  
  // Initial message
  res.write(`data: ${JSON.stringify({ type: 'connected' })}\n\n`)
  
  // เก็บ connection
  clients.set(userId, res)
  
  // Heartbeat ทุก 30 วินาที
  const heartbeat = setInterval(() => {
    res.write(': heartbeat\n\n')
  }, 30000)
  
  // Cleanup เมื่อ client disconnect
  req.on('close', () => {
    clearInterval(heartbeat)
    clients.delete(userId)
    console.log(`Client ${userId} disconnected`)
  })
})

// ส่ง event ไปยัง specific user
function sendEventToUser(userId: string, event: string, data: any) {
  const client = clients.get(userId)
  if (client) {
    client.write(`event: ${event}\n`)
    client.write(`data: ${JSON.stringify(data)}\n\n`)
  }
}

// ส่ง event ไปทุกคน
function broadcastEvent(event: string, data: any) {
  clients.forEach((client) => {
    client.write(`event: ${event}\n`)
    client.write(`data: ${JSON.stringify(data)}\n\n`)
  })
}
```

### EventSource ใน Browser

```typescript
// hooks/useSSE.ts
import { useEffect, useState, useRef } from 'react'

interface SSEOptions {
  url: string
  onMessage?: (data: any) => void
  onError?: (error: Event) => void
  onOpen?: () => void
  events?: {
    [eventName: string]: (data: any) => void
  }
}

export function useSSE({
  url,
  onMessage,
  onError,
  onOpen,
  events = {},
}: SSEOptions) {
  const [isConnected, setIsConnected] = useState(false)
  const [error, setError] = useState<string | null>(null)
  const eventSourceRef = useRef<EventSource | null>(null)
  
  useEffect(() => {
    const eventSource = new EventSource(url, {
      withCredentials: true,
    })
    
    eventSourceRef.current = eventSource
    
    eventSource.onopen = () => {
      setIsConnected(true)
      setError(null)
      onOpen?.()
    }
    
    eventSource.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data)
        onMessage?.(data)
      } catch {
        onMessage?.(event.data)
      }
    }
    
    eventSource.onerror = (event) => {
      setIsConnected(false)
      setError('Connection error')
      onError?.(event)
    }
    
    // Custom event listeners
    Object.entries(events).forEach(([eventName, handler]) => {
      eventSource.addEventListener(eventName, (event) => {
        try {
          const data = JSON.parse((event as MessageEvent).data)
          handler(data)
        } catch {
          handler((event as MessageEvent).data)
        }
      })
    })
    
    return () => {
      eventSource.close()
    }
  }, [url])
  
  return { isConnected, error, eventSource: eventSourceRef.current }
}
```

---

## Step 1531-1540: Next.js Route Handler + SSE {#nextjs-route-handler-sse}

Next.js App Router รองรับ SSE ผ่าน Route Handlers

```typescript
// app/api/notifications/stream/route.ts
import { NextRequest } from 'next/server'
import { getServerSession } from 'next-auth'
import { authOptions } from '@/lib/auth'

// Store active connections
const connections = new Map<string, ReadableStreamDefaultController>()

export async function GET(request: NextRequest) {
  const session = await getServerSession(authOptions)
  
  if (!session?.user) {
    return new Response('Unauthorized', { status: 401 })
  }
  
  const userId = session.user.id
  
  // สร้าง ReadableStream
  const stream = new ReadableStream({
    start(controller) {
      // เก็บ connection
      connections.set(userId, controller)
      
      // ส่ง initial message
      const data = JSON.stringify({ type: 'connected', userId })
      controller.enqueue(`data: ${data}\n\n`)
      
      // ตั้ง heartbeat
      const heartbeat = setInterval(() => {
        try {
          controller.enqueue(': heartbeat\n\n')
        } catch {
          clearInterval(heartbeat)
        }
      }, 30000)
      
      // Cleanup
      request.signal.addEventListener('abort', () => {
        clearInterval(heartbeat)
        connections.delete(userId)
        controller.close()
      })
    },
  })
  
  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
      'Access-Control-Allow-Origin': '*',
    },
  })
}

// Export function สำหรับส่ง notification
export function sendNotification(userId: string, notification: any) {
  const controller = connections.get(userId)
  if (controller) {
    const data = JSON.stringify(notification)
    controller.enqueue(`event: notification\ndata: ${data}\n\n`)
  }
}

// Broadcast ไปทุกคน
export function broadcastNotification(notification: any) {
  const data = JSON.stringify(notification)
  connections.forEach((controller) => {
    controller.enqueue(`event: notification\ndata: ${data}\n\n`)
  })
}
```

### AI Streaming ด้วย SSE

```typescript
// app/api/chat/route.ts
import { NextRequest } from 'next/server'
import Anthropic from '@anthropic-ai/sdk'

const client = new Anthropic()

export async function POST(request: NextRequest) {
  const { messages } = await request.json()
  
  // สร้าง streaming response
  const stream = new ReadableStream({
    async start(controller) {
      const encoder = new TextEncoder()
      
      const messageStream = await client.messages.stream({
        model: 'claude-opus-4-5',
        max_tokens: 1024,
        messages,
      })
      
      for await (const event of messageStream) {
        if (event.type === 'content_block_delta') {
          const text = event.delta.text
          const data = JSON.stringify({ type: 'text', content: text })
          controller.enqueue(encoder.encode(`data: ${data}\n\n`))
        }
        
        if (event.type === 'message_stop') {
          controller.enqueue(encoder.encode('data: [DONE]\n\n'))
          controller.close()
        }
      }
    },
  })
  
  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
    },
  })
}
```

```typescript
// hooks/useStreamingChat.ts
'use client'

import { useState, useCallback } from 'react'

interface Message {
  role: 'user' | 'assistant'
  content: string
}

export function useStreamingChat() {
  const [messages, setMessages] = useState<Message[]>([])
  const [isStreaming, setIsStreaming] = useState(false)
  const [streamingContent, setStreamingContent] = useState('')
  
  const sendMessage = useCallback(async (content: string) => {
    const userMessage: Message = { role: 'user', content }
    const newMessages = [...messages, userMessage]
    setMessages(newMessages)
    setIsStreaming(true)
    setStreamingContent('')
    
    try {
      const response = await fetch('/api/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ messages: newMessages }),
      })
      
      const reader = response.body!.getReader()
      const decoder = new TextDecoder()
      let buffer = ''
      let fullContent = ''
      
      while (true) {
        const { done, value } = await reader.read()
        if (done) break
        
        buffer += decoder.decode(value, { stream: true })
        
        // Parse SSE lines
        const lines = buffer.split('\n')
        buffer = lines.pop() || ''
        
        for (const line of lines) {
          if (line.startsWith('data: ')) {
            const data = line.slice(6)
            
            if (data === '[DONE]') {
              setMessages((prev) => [
                ...prev,
                { role: 'assistant', content: fullContent },
              ])
              setStreamingContent('')
              break
            }
            
            try {
              const parsed = JSON.parse(data)
              if (parsed.type === 'text') {
                fullContent += parsed.content
                setStreamingContent(fullContent)
              }
            } catch {}
          }
        }
      }
    } catch (error) {
      console.error('Error:', error)
    } finally {
      setIsStreaming(false)
    }
  }, [messages])
  
  return {
    messages,
    isStreaming,
    streamingContent,
    sendMessage,
  }
}
```

---

## Step 1541-1545: Real-time Chat App {#real-time-chat-app}

### Chat UI Component

```typescript
// components/ChatRoom.tsx
'use client'

import { useRef, useEffect, useState } from 'react'
import { useChat } from '@/hooks/useChat'
import { useSession } from 'next-auth/react'

interface ChatRoomProps {
  roomId: string
  roomName: string
}

export function ChatRoom({ roomId, roomName }: ChatRoomProps) {
  const { data: session } = useSession()
  const { messages, typingUsers, isLoading, sendMessage, startTyping } = useChat(roomId)
  const [inputValue, setInputValue] = useState('')
  const messagesEndRef = useRef<HTMLDivElement>(null)
  
  // Auto scroll ไปที่ล่างสุด
  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' })
  }, [messages, typingUsers])
  
  function handleSubmit(e: React.FormEvent) {
    e.preventDefault()
    if (!inputValue.trim()) return
    
    sendMessage(inputValue)
    setInputValue('')
  }
  
  function handleKeyPress(e: React.ChangeEvent<HTMLInputElement>) {
    setInputValue(e.target.value)
    startTyping()
  }
  
  return (
    <div className="flex flex-col h-screen max-h-[600px] bg-white rounded-xl shadow-lg">
      {/* Header */}
      <div className="flex items-center px-4 py-3 border-b bg-gray-50 rounded-t-xl">
        <div className="w-3 h-3 bg-green-500 rounded-full mr-2" />
        <h2 className="font-semibold text-gray-800">{roomName}</h2>
      </div>
      
      {/* Messages */}
      <div className="flex-1 overflow-y-auto p-4 space-y-4">
        {isLoading ? (
          <div className="flex justify-center items-center h-full">
            <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600" />
          </div>
        ) : (
          <>
            {messages.map((message) => {
              const isOwn = message.sender.id === session?.user?.id
              
              return (
                <div
                  key={message.id}
                  className={`flex items-start gap-2 ${
                    isOwn ? 'flex-row-reverse' : 'flex-row'
                  }`}
                >
                  <img
                    src={message.sender.avatar || '/default-avatar.png'}
                    alt={message.sender.name}
                    className="w-8 h-8 rounded-full flex-shrink-0"
                  />
                  <div className={`max-w-xs ${isOwn ? 'items-end' : 'items-start'} flex flex-col`}>
                    {!isOwn && (
                      <span className="text-xs text-gray-500 mb-1">
                        {message.sender.name}
                      </span>
                    )}
                    <div
                      className={`px-4 py-2 rounded-2xl text-sm ${
                        isOwn
                          ? 'bg-blue-600 text-white rounded-tr-none'
                          : 'bg-gray-100 text-gray-800 rounded-tl-none'
                      }`}
                    >
                      {message.content}
                    </div>
                    <span className="text-xs text-gray-400 mt-1">
                      {new Date(message.timestamp).toLocaleTimeString('th-TH', {
                        hour: '2-digit',
                        minute: '2-digit',
                      })}
                    </span>
                  </div>
                </div>
              )
            })}
            
            {/* Typing Indicator */}
            {typingUsers.length > 0 && (
              <div className="flex items-center gap-2 text-sm text-gray-500">
                <div className="flex gap-1">
                  <span className="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style={{ animationDelay: '0ms' }} />
                  <span className="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style={{ animationDelay: '150ms' }} />
                  <span className="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style={{ animationDelay: '300ms' }} />
                </div>
                <span>
                  {typingUsers.map((u) => u.name).join(', ')} กำลังพิมพ์...
                </span>
              </div>
            )}
            
            <div ref={messagesEndRef} />
          </>
        )}
      </div>
      
      {/* Input */}
      <form onSubmit={handleSubmit} className="p-4 border-t">
        <div className="flex gap-2">
          <input
            type="text"
            value={inputValue}
            onChange={handleKeyPress}
            placeholder="พิมพ์ข้อความ..."
            className="flex-1 px-4 py-2 rounded-full border border-gray-200 focus:outline-none focus:border-blue-500"
          />
          <button
            type="submit"
            disabled={!inputValue.trim()}
            className="w-10 h-10 bg-blue-600 text-white rounded-full flex items-center justify-center disabled:opacity-50 hover:bg-blue-700 transition"
          >
            →
          </button>
        </div>
      </form>
    </div>
  )
}
```

---

## Step 1546-1548: Live Notifications {#live-notifications}

```typescript
// components/NotificationBell.tsx
'use client'

import { useState, useEffect } from 'react'
import { useSSE } from '@/hooks/useSSE'
import { Bell } from 'lucide-react'

interface Notification {
  id: string
  type: 'message' | 'like' | 'comment' | 'follow'
  title: string
  body: string
  isRead: boolean
  createdAt: string
  link?: string
}

export function NotificationBell() {
  const [notifications, setNotifications] = useState<Notification[]>([])
  const [isOpen, setIsOpen] = useState(false)
  
  const unreadCount = notifications.filter((n) => !n.isRead).length
  
  // Connect to SSE
  useSSE({
    url: '/api/notifications/stream',
    events: {
      notification: (data: Notification) => {
        setNotifications((prev) => [data, ...prev])
        
        // Browser notification
        if ('Notification' in window && Notification.permission === 'granted') {
          new Notification(data.title, {
            body: data.body,
            icon: '/notification-icon.png',
          })
        }
      },
    },
  })
  
  // โหลด notifications เก่า
  useEffect(() => {
    fetch('/api/notifications')
      .then((res) => res.json())
      .then((data) => setNotifications(data.notifications))
  }, [])
  
  async function markAllAsRead() {
    await fetch('/api/notifications/read-all', { method: 'POST' })
    setNotifications((prev) =>
      prev.map((n) => ({ ...n, isRead: true }))
    )
  }
  
  return (
    <div className="relative">
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="relative p-2 rounded-full hover:bg-gray-100"
        aria-label={`${unreadCount} การแจ้งเตือนใหม่`}
      >
        <Bell className="w-5 h-5" />
        {unreadCount > 0 && (
          <span className="absolute -top-1 -right-1 bg-red-500 text-white text-xs rounded-full w-5 h-5 flex items-center justify-center">
            {unreadCount > 9 ? '9+' : unreadCount}
          </span>
        )}
      </button>
      
      {isOpen && (
        <div className="absolute right-0 mt-2 w-80 bg-white rounded-xl shadow-lg border z-50">
          <div className="flex items-center justify-between p-4 border-b">
            <h3 className="font-semibold">การแจ้งเตือน</h3>
            {unreadCount > 0 && (
              <button
                onClick={markAllAsRead}
                className="text-sm text-blue-600 hover:underline"
              >
                อ่านทั้งหมด
              </button>
            )}
          </div>
          
          <div className="max-h-80 overflow-y-auto">
            {notifications.length === 0 ? (
              <p className="text-center text-gray-500 py-8">
                ไม่มีการแจ้งเตือน
              </p>
            ) : (
              notifications.map((notification) => (
                <div
                  key={notification.id}
                  className={`p-4 border-b hover:bg-gray-50 cursor-pointer ${
                    !notification.isRead ? 'bg-blue-50' : ''
                  }`}
                >
                  <p className="font-medium text-sm">{notification.title}</p>
                  <p className="text-sm text-gray-600">{notification.body}</p>
                  <p className="text-xs text-gray-400 mt-1">
                    {new Date(notification.createdAt).toLocaleString('th-TH')}
                  </p>
                </div>
              ))
            )}
          </div>
        </div>
      )}
    </div>
  )
}
```

---

## Step 1549-1550: Collaborative Features {#collaborative-features}

```typescript
// hooks/useCollaborativeEditor.ts
import { useEffect, useState, useCallback } from 'react'
import { useSocket } from '@/contexts/SocketContext'

interface CollaborativeUser {
  id: string
  name: string
  color: string
  cursor?: {
    line: number
    column: number
  }
}

export function useCollaborativeEditor(documentId: string) {
  const { socket } = useSocket()
  const [content, setContent] = useState('')
  const [collaborators, setCollaborators] = useState<CollaborativeUser[]>([])
  
  useEffect(() => {
    if (!socket) return
    
    socket.emit('document:join', documentId)
    
    socket.on('document:content', (data: { content: string }) => {
      setContent(data.content)
    })
    
    socket.on('document:change', (data: {
      userId: string
      changes: Array<{ type: 'insert' | 'delete'; position: number; text?: string; length?: number }>
    }) => {
      // Apply Operational Transformation (OT)
      setContent((prev) => applyChanges(prev, data.changes))
    })
    
    socket.on('collaborators:update', (users: CollaborativeUser[]) => {
      setCollaborators(users)
    })
    
    socket.on('cursor:update', (data: {
      userId: string
      cursor: { line: number; column: number }
    }) => {
      setCollaborators((prev) =>
        prev.map((u) =>
          u.id === data.userId ? { ...u, cursor: data.cursor } : u
        )
      )
    })
    
    return () => {
      socket.emit('document:leave', documentId)
    }
  }, [socket, documentId])
  
  const updateContent = useCallback((newContent: string, changes: any[]) => {
    setContent(newContent)
    socket?.emit('document:change', {
      documentId,
      changes,
    })
  }, [socket, documentId])
  
  const updateCursor = useCallback((cursor: { line: number; column: number }) => {
    socket?.emit('cursor:update', { documentId, cursor })
  }, [socket, documentId])
  
  return { content, collaborators, updateContent, updateCursor }
}

function applyChanges(content: string, changes: any[]): string {
  let result = content
  // Simple implementation - ควรใช้ library เช่น yjs หรือ sharedb
  for (const change of changes) {
    if (change.type === 'insert') {
      result = result.slice(0, change.position) + change.text + result.slice(change.position)
    } else if (change.type === 'delete') {
      result = result.slice(0, change.position) + result.slice(change.position + change.length)
    }
  }
  return result
}
```

---

## 🧪 Quiz - Part 48

**ข้อ 1:** ความแตกต่างหลักระหว่าง WebSocket และ SSE คือ?
- A) WebSocket เร็วกว่า SSE
- B) WebSocket รองรับการสื่อสารสองทาง SSE รองรับเฉพาะ Server → Client
- C) SSE ใช้ TCP WebSocket ใช้ UDP
- D) WebSocket ใช้ HTTP SSE ใช้ WebSocket Protocol

**ข้อ 2:** ใน Socket.io การส่ง event ไปยังทุกคนในห้องยกเว้นผู้ส่งคือ?
- A) `socket.emit('event', data)`
- B) `io.to(roomId).emit('event', data)`
- C) `socket.to(roomId).emit('event', data)`
- D) `socket.broadcast.to(roomId).emit('event', data)`

**ข้อ 3:** Content-Type สำหรับ SSE response คือ?
- A) `application/json`
- B) `text/plain`
- C) `text/event-stream`
- D) `application/stream`

**ข้อ 4:** SSE data format ที่ถูกต้องคือ?
- A) `data: Hello World`
- B) `data: Hello World\n\n`
- C) `{ data: "Hello World" }`
- D) `SSE: Hello World`

**เฉลย:** 1-B, 2-C, 3-C, 4-B

---

> **➡️ Next:** [Part 49: Advanced Patterns](./part-49-advanced-patterns.md)
