# Add these to implement Replay feature in chat
### Chat.tsx
```typescript
import React, { useEffect, useState, useRef } from 'react';
import { useNavigate, useParams } from 'react-router-dom';
import { io, Socket } from 'socket.io-client';
import { useDispatch, useSelector } from 'react-redux';
import { checkTutorAuthStatus } from '../../features/tutor/tutorSlice';
import { RootState } from '../store/store';
import { apiService } from '../../services/api';

interface Message {
  _id: string;
  sender: { _id: string; name: string };
  senderRole: 'Student' | 'Tutor';
  content: string;
  timestamp: string;
  isRead: boolean;
  replyTo?: string;
}

interface ChatHistoryResponse {
  chat: {
    messages: Message[];
  };
  userName: string;
  tutorName: string;
}

const Chat: React.FC = () => {
  const [messages, setMessages] = useState<Message[]>([]);
  const [inputMessage, setInputMessage] = useState('');
  const [socket, setSocket] = useState<Socket | null>(null);
  const { roomId } = useParams<{ roomId: string }>();
  const [userName, setUserName] = useState<string | null>(null);
  const [tutorName, setTutorName] = useState<string | null>(null);
  const [replyingTo, setReplyingTo] = useState<Message | null>(null);
  const dispatch = useDispatch();
  const navigate = useNavigate();

  const user = useSelector((state: RootState) => state.auth.user);
  const tutor = useSelector((state: RootState) => state.tutor.tutor);

  const messagesEndRef = useRef<HTMLDivElement>(null);
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    dispatch(checkTutorAuthStatus());
  }, [dispatch]);

  const getUserRole = () => {
    if (!roomId) return 'Unknown';
    const [studentId, tutorId] = roomId.split('_');
    if (user?.id === studentId) return 'Student';
    if (tutor?.id === tutorId) return 'Tutor';
    return 'Unknown';
  };

  const userRole = getUserRole();
  const participant = userRole === 'Student' ? user : userRole === 'Tutor' ? tutor : null;
  if(!participant){
    navigate('/notFound')
  }

  useEffect(() => {
    const socketServerUrl = import.meta.env.VITE_BASE_URL; 
    const newSocket = io(socketServerUrl, {
      withCredentials: true,
      transports: ['websocket']
    }); 
    newSocket.on('connect_error', (error) => {
      console.error('Socket connection error:', error);
    });
    setSocket(newSocket);

    return () => {
      newSocket.disconnect();
    };
  }, []);

  useEffect(() => {
    if (socket && roomId && participant) {
      socket.emit('join_room', roomId);
      console.log(`Joined room: ${roomId}`);
      
      socket.on('receive_message', (message: Message) => {
        console.log('Received message:', message);
        setMessages((prevMessages) => [...prevMessages, message]);
        if (message.sender._id !== participant.id) {
          socket.emit('mark_messages_as_read', { roomId, userId: participant.id });
        }
      });

      socket.on('messages_marked_as_read', (userId: string) => {
        if (userId !== participant.id) {
          setMessages((prevMessages) =>
            prevMessages.map((msg) =>
              msg.sender._id === participant.id ? { ...msg, isRead: true } : msg
            )
          );
        }
      });

      fetchChatHistory(roomId);
    } else {
      console.log('Socket or participant is missing');
    }
  }, [socket, roomId, participant]);

  useEffect(() => {
    scrollToBottom();
  }, [messages]);

  const fetchChatHistory = async (roomId: string) => {
    try {
      const response = await apiService.get<ChatHistoryResponse>(`/chat/history/${roomId}`);
      setMessages(response.chat.messages);
      setUserName(response.userName);
      setTutorName(response.tutorName);
      if (socket && participant) {
        socket.emit('mark_messages_as_read', { roomId, userId: participant.id });
      }
    } catch (error) {
      console.error('Error fetching chat history:', error);
    }
  };

  const sendMessage = () => {
    if (socket && inputMessage.trim() !== '' && participant) {
      const messageData = { 
        roomId, 
        senderId: participant.id, 
        content: inputMessage,
        senderRole: userRole,
        replyTo: replyingTo?._id  
      };
      
      setInputMessage('');
      setReplyingTo(null);  
  
      console.log('Sending message:', messageData);
      socket.emit('send_message', messageData, (error: any) => {
        if (error) {
          console.error('Error sending message:', error);
          setInputMessage(inputMessage);
        }
      });
    }
  };
  
  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };

  const handleReply = (message: Message) => {
    setReplyingTo(message);
    inputRef.current?.focus();
  };

  const cancelReply = () => {
    setReplyingTo(null);
  };

  const renderMessage = (message: Message) => {
    const isOwnMessage = message.sender._id === participant?.id;
    const repliedMessage = message.replyTo ? messages.find(m => m._id === message.replyTo) : null;

    return (
      <div
        key={message._id}
        className={`flex ${isOwnMessage ? 'justify-end' : 'justify-start'}`}
      >
        <div
          className={`max-w-xs p-3 rounded-lg shadow-md ${
            isOwnMessage
              ? 'bg-blue-600 text-white rounded-br-none'
              : 'bg-gray-200 text-gray-900 rounded-bl-none'
          }`}
        >
          {repliedMessage && (
            <div className="text-xs italic mb-2 p-2 bg-opacity-20 bg-gray-500 rounded">
              <p className="font-semibold">{repliedMessage.sender.name}:</p>
              <p>{repliedMessage.content.substring(0, 50)}...</p>
            </div>
          )}
          <p className="text-sm font-semibold">
            {message.sender.name} ({message.senderRole})
          </p>
          <p className="mt-1">{message.content}</p>
          <p className="text-xs mt-2 text-gray-400">
            {new Date(message.timestamp).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}
            {isOwnMessage && (
              <span className="ml-2">
                {message.isRead ? '✓✓' : '✓'}
              </span>
            )}
          </p>
          <button 
            onClick={() => handleReply(message)}
            className="text-xs underline mt-1 hover:text-gray-300"
          >
            Reply
          </button>
        </div>
      </div>
    );
  };

  return (
    <div className="flex flex-col h-screen bg-gray-100">
      {/* Header */}
      <div className="bg-gradient-to-r from-indigo-900 to-purple-800 text-white p-4 shadow-lg">
        <h1 className="text-3xl font-bold mb-2">
          Chat: {userName || 'Send a message to connect with the Tutor'} &ndash; {tutorName || 'Wait...'}
        </h1>
        <p className="text-sm">Logged in as: {participant?.name || 'Unknown'} ({userRole})</p>
        <p className="text-sm">Chatting with: {userRole === "Student" ? tutorName : userName}</p>
      </div>

      {/* Messages */}
      <div className="flex-1 overflow-y-auto p-6 space-y-4 bg-gray-50">
        {messages.map(renderMessage)}
        <div ref={messagesEndRef} />
      </div>
      {/* Input */}
      <div className="p-4 bg-white border-t shadow-md">
        {replyingTo && (
          <div className="mb-2 p-2 bg-gray-100 rounded flex justify-between items-center">
            <span className="text-sm">
              Replying to: {replyingTo.content.substring(0, 30)}...
            </span>
            <button onClick={cancelReply} className="text-red-500 hover:text-red-700">
              ✕
            </button>
          </div>
        )}
        <div className="flex items-center">
          <input
            type="text"
            value={inputMessage}
            onChange={(e) => setInputMessage(e.target.value)}
            onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
            className="flex-1 border border-gray-300 rounded-l-lg p-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
            placeholder="Type a message..."
            ref={inputRef}
          />
          <button
            onClick={sendMessage}
            className="bg-blue-500 hover:bg-blue-600 text-white px-6 py-2 rounded-r-lg focus:outline-none"
          >
            Send
          </button>
        </div>
      </div>
    </div>
  );
};

export default Chat;
```

### socketHandler.ts
```typescript

// src/sockets/socketHandler.ts

import { Server, Socket } from 'socket.io';
import Chat from '../models/Chat';
import User from '../models/User';
import Tutor from '../models/Tutor';

export function initializeSocket(io: Server) {
  io.on('connection', (socket: Socket) => {
    console.log('A user connected');

    socket.on('join_room', (roomId: string) => {
      socket.join(roomId);
      console.log(`User joined room: ${roomId}`);
    });

    socket.on('send_message', async ({ roomId, senderId, content, senderRole,replyTo }) => {
      try {
        console.log('Message received on server:', { roomId, senderId, content, senderRole,replyTo });

        let chat = await Chat.findOne({ roomId });
        
        if (!chat) {
          console.log('Chat not found! Creating new chat room.');
          chat = new Chat({
            roomId,
            participants: [senderId],
            messages: [],
          });
          await chat.save();
          console.log('New chat room created:', chat);
        }

        let sender;
        if (senderRole === 'Student') {
          sender = await User.findById(senderId);
        } else if (senderRole === 'Tutor') {
          sender = await Tutor.findById(senderId);
        }

        if (!sender) {
          console.log(`${senderRole} not found`);
          throw new Error(`${senderRole} not found`);
        }

        const newMessage = {
          sender: senderId,
          senderRole,
          content,
          timestamp: new Date(),
          isRead: false ,
          replyTo: replyTo || null
        };
        console.log("newMessage", newMessage);
        chat.messages.push(newMessage);
        await chat.save();

        console.log('Message saved to DB:', newMessage);

        const populatedMessage = {
          ...newMessage,
          sender: { _id: sender._id, name: sender.name },
        };
        io.to(roomId).emit('receive_message', populatedMessage);
        console.log(`Message emitted to room: ${roomId}`, populatedMessage);
      } catch (error) {
        console.error('Error saving message:', error);
      }
    });

    socket.on('mark_messages_as_read', async ({ roomId, userId }) => {
      try {
        const chat = await Chat.findOne({ roomId });
        if (chat) {
          let updated = false;
          chat.messages.forEach(message => {
            if (message.sender.toString() !== userId && !message.isRead) {
              message.isRead = true;
              updated = true;
            }
          });
          if (updated) {
            await chat.save();
            io.to(roomId).emit('messages_marked_as_read', userId);
          }
        }
      } catch (error) {
        console.error('Error marking messages as read:', error);
      }
    });

    socket.on('disconnect', () => {
      console.log('A user disconnected');
    });
  });
}
```
### Chat model.ts:-

```typescript
// models/Chat.ts
import mongoose, { Document, Schema } from 'mongoose';

interface IMessage {
  sender: mongoose.Types.ObjectId;
  senderRole: 'Student' | 'Tutor';
  content: string;
  timestamp: Date;
  isRead: boolean;
  replyTo?: mongoose.Types.ObjectId | string; 
}

interface IChat extends Document {
  roomId: string;
  participants: mongoose.Types.ObjectId[];
  messages: IMessage[];
}

const ChatSchema: Schema = new Schema({
  roomId: { type: String, required: true, unique: true },
  participants: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
  messages: [{
    sender: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
    senderRole: { type: String, enum: ['Student', 'Tutor'], required: true },
    content: { type: String, required: true },
    timestamp: { type: Date, default: Date.now },
    isRead: { type: Boolean, default: false } ,
    replyTo: { type: Schema.Types.Mixed, required: false }
  }]
});

export default mongoose.model<IChat>('Chat', ChatSchema);
```
