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

## 🎵 Hidden Harmony - Anonymous Dating App

### Frontend Development Role
Led frontend development for an innovative anonymous dating app using Android (Java). Implemented interest-based matching, real-time notifications, and profile management with a focus on user privacy and seamless experience.

### Key Contributions
- Built interest-based matching algorithm and UI
- Developed background notification service
- Created comprehensive profile management
- Implemented matching interface with swipe functionality
- Integrated WebSocket for real-time match notifications

### Technical Implementation
- **Platform**: Android SDK with Java
- **Communication**: WebSocket for real-time notifications
- **Architecture**: Clean Architecture principles
- **UI/UX**: Custom Android components
- **Services**: Background notification service
- **Testing**: JUnit and Espresso

### Code Samples

```java
// Real-time Notification Service
public class NotificationService extends Service implements WebSocketListener {
    private int notificationId = 0;
    private int userId = -1;

    @Override
    public void onCreate() {
        super.onCreate();
        Profile userProfile = UserProfileManager.getInstance().getUserProfile();
        if (userProfile != null) {
            userId = userProfile.getId();
            
            // Create notification channel for Android O and above
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                NotificationChannel channel = new NotificationChannel(
                    "NotificationChannel", 
                    "channel", 
                    NotificationManager.IMPORTANCE_DEFAULT
                );
                NotificationManager manager = getSystemService(NotificationManager.class);
                manager.createNotificationChannel(channel);
            }

            // Connect to WebSocket for real-time notifications
            WebSocketManager.getInstance().connectWebSocket(
                "ws://coms-309-054.class.las.iastate.edu:8080/notifications/" + userId
            );
            WebSocketManager.getInstance().setWebSocketListener(this);
        }
    }

    @Override
    public void onWebSocketMessage(String message) {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(
            NotificationService.this, 
            "NotificationChannel"
        );
        builder.setContentTitle("Hidden Harmony");
        builder.setContentText(message);
        builder.setSmallIcon(R.drawable.ic_launcher_background);
        builder.setAutoCancel(true);

        NotificationManagerCompat managerCompat = NotificationManagerCompat.from(
            NotificationService.this
        );
        
        if (hasNotificationPermission()) {
            managerCompat.notify(notificationId++, builder.build());
        }
    }
}

// Profile Management with Interest-Based Matching
public class Profile {
    private int id;
    private String name;
    private String bio;
    private ArrayList<String> interests;
    private ArrayList<Profile> potentialMatches;

    public void updatePotentialMatches(Context context, UpdateMatchesCallback callback) {
        String apiUrl = "http://coms-309-054.class.las.iastate.edu:8080/api/matches/potential-matches/" + this.id;
        potentialMatches.clear();
        
        HTTPHelper httpHelper = new HTTPHelper(context, apiUrl);
        httpHelper.getArrayOfInts(apiUrl, new HTTPHelper.VolleyArrayCallback() {
            @Override
            public void onSuccess(JSONArray response) {
                // Process potential matches based on interests
                for (int i = 0; i < response.length(); i++) {
                    try {
                        int profileId = response.getInt(i);
                        Profile profile = new Profile(profileId, context, 
                            profile -> {
                                potentialMatches.add(profile);
                                if (potentialMatches.size() == response.length()) {
                                    callback.onMatchesUpdated(potentialMatches);
                                }
                            });
                    } catch (JSONException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
    }
}
```

### Development Practices
- Agile methodology with GitLab CI/CD
- Code review process
- Comprehensive documentation
- Unit and integration testing
- Security best practices

## 🛠️ Development Practices

- TypeScript for type safety
- ESLint + Prettier configuration
- Git workflow with feature branches
- Code review process
- Continuous integration setup 