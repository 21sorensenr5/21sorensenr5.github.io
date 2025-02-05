# ProsOutdoors - Technical Showcase

## 🎯 Project Overview

A full-stack video sharing platform built for outdoor sports enthusiasts. This project demonstrates implementation of modern web development practices, cloud integration, and secure user management.

## 🛠️ Technical Implementation

### Authentication System
```typescript
// JWT-based authentication middleware
const authMiddleware = async (req: Request, res: Response, next: NextFunction) => {
  try {
    // Extract token from Authorization header
    const token = req.headers.authorization?.split(' ')[1];
    
    // Verify and decode token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // Attach user to request
    req.user = await User.findById(decoded.userId);
    next();
  } catch (error) {
    res.status(401).json({ message: 'Authentication failed' });
  }
};
```

### Video Processing
```typescript
// Video upload with automatic thumbnail generation
const uploadVideo = async (req: Request, res: Response) => {
  try {
    const result = await cloudinary.uploader.upload(req.file.path, {
      resource_type: 'video',
      folder: 'videos',
      eager: [{ 
        format: 'jpg',
        transformation: [
          { width: 400, height: 300, crop: "fill" }
        ]
      }]
    });

    const video = new Video({
      title: req.body.title,
      url: result.secure_url,
      thumbnailUrl: result.eager[0].secure_url,
      user: req.user._id
    });

    await video.save();
    res.status(201).json(video);
  } catch (error) {
    res.status(500).json({ message: 'Upload failed' });
  }
};
```

### React Components & Custom Hooks
```typescript
// Custom hook for video upload with progress tracking
const useVideoUpload = () => {
  const [progress, setProgress] = useState(0);
  const [error, setError] = useState<string | null>(null);

  const uploadVideo = async (file: File, metadata: VideoMetadata) => {
    try {
      const formData = new FormData();
      formData.append('video', file);
      formData.append('metadata', JSON.stringify(metadata));

      const response = await axios.post('/api/videos/upload', formData, {
        onUploadProgress: (progressEvent) => {
          const percentage = Math.round(
            (progressEvent.loaded * 100) / progressEvent.total
          );
          setProgress(percentage);
        }
      });

      return response.data;
    } catch (err) {
      setError('Upload failed');
      throw err;
    }
  };

  return { uploadVideo, progress, error };
};

// Video player component with custom controls
const VideoPlayer: React.FC<VideoPlayerProps> = ({ video }) => {
  const [isPlaying, setIsPlaying] = useState(false);
  const videoRef = useRef<HTMLVideoElement>(null);

  const togglePlay = () => {
    if (videoRef.current) {
      if (isPlaying) {
        videoRef.current.pause();
      } else {
        videoRef.current.play();
      }
      setIsPlaying(!isPlaying);
    }
  };

  return (
    <div className="video-player">
      <video
        ref={videoRef}
        src={video.url}
        poster={video.thumbnailUrl}
        onEnded={() => setIsPlaying(false)}
      />
      <div className="controls">
        <Button onClick={togglePlay}>
          {isPlaying ? <PauseIcon /> : <PlayIcon />}
        </Button>
        <VideoProgress video={videoRef} />
      </div>
    </div>
  );
};
```

### State Management with Context
```typescript
// Auth context for global user state management
interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType>(null!);

export const AuthProvider: React.FC = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string) => {
    try {
      const response = await axios.post('/api/auth/login', {
        email,
        password
      });
      
      const { token, user } = response.data;
      localStorage.setItem('token', token);
      setUser(user);
    } catch (error) {
      throw new Error('Authentication failed');
    }
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};
```

## 🏗️ Architecture Highlights

### Frontend Architecture
- React with TypeScript for type safety
- Material-UI components with custom styling
- Context API for state management
- Custom hooks for business logic
- Responsive design implementation

### Backend Architecture
- Node.js/Express RESTful API
- MongoDB with Mongoose ODM
- JWT authentication flow
- Cloud storage integration
- Error handling middleware

## 💡 Key Technical Features

### Video Processing
- Automatic thumbnail generation
- Optimized video streaming
- Progress tracking
- Format validation

### User Management
- Secure password hashing
- JWT token rotation
- Profile customization
- Following system

### Performance Optimizations
- Lazy loading of components
- Image optimization
- Caching strategies
- Efficient database queries

## 🔒 Security Implementation

- JWT token-based authentication
- Password hashing with bcrypt
- XSS protection
- CORS configuration
- Input validation
- File type verification

## 📱 Responsive Design

The application is fully responsive across devices:
- Desktop optimization
- Tablet-friendly layouts
- Mobile-first approach
- Touch-friendly interactions

## 🛠️ Development Practices

- TypeScript for type safety
- ESLint + Prettier configuration
- Git workflow with feature branches
- Code review process
- Continuous integration setup 