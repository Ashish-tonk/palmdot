import React, { useState, useEffect, createContext, useContext } from 'react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged, createUserWithEmailAndPassword, signInWithEmailAndPassword, signOut } from 'firebase/auth';
import { getFirestore, doc, setDoc, onSnapshot, collection } from 'firebase/firestore';

// --- Firebase Configuration and Initialization ---
// These global variables are provided by the Canvas environment.
// We provide fallbacks for local development.
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

// --- Contexts for Global State ---
const AuthContext = createContext(null);
const CourseContext = createContext(null);

// --- Dummy Course Data ---
// In a real application, this might come from a database.
const courseContent = [
  {
    id: 'topic1',
    title: 'Foundations of Skincare',
    description: 'Learn the essential steps for a healthy, radiant complexion, understanding your skin type, and building a basic routine.',
    videos: [
      { id: 'v1_1', title: 'Understanding Your Skin Type', videoUrl: 'https://www.youtube.com/watch?v=ZzNPZ5v5bTY', description: 'Identify your unique skin type (oily, dry, combination, normal, sensitive) to tailor your skincare approach effectively.', notes: 'Key takeaways: Observe T-zone oiliness, post-wash tightness, redness. Product focus: Gentle cleansers, non-comedogenic moisturizers.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
      { id: 'v1_2', title: 'The Basic Skincare Routine', videoUrl: 'https://www.youtube.com/embed/dQw4w9WgXcQ?si=RjZ39l22-0mS-1', description: 'Master the core steps: cleanse, treat, moisturize, and protect. Consistency is key!', notes: 'AM: Cleanser, Vitamin C, Moisturizer, SPF. PM: Cleanser, Treatment (Retinol/AHA), Moisturizer. Specifications: Use lukewarm water, pat dry, apply products to damp skin.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
      { id: 'v1_3', title: 'Hydration & Barrier Health', videoUrl: 'https://www.youtube.com/embed/dQw4w9WgXcQ?si=RjZ39l22-0mS-1', description: 'Discover how to maintain a strong skin barrier and deep hydration for plump, resilient skin.', notes: 'Ingredients to look for: Hyaluronic Acid, Ceramides, Glycerin. Avoid over-exfoliation. Specifications: Drink plenty of water, use a humidifier.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
    ]
  },
  {
    id: 'topic2',
    title: 'Hair Optimization',
    description: 'Unlock the secrets to healthier hair, from proper washing techniques to styling for your face shape.',
    videos: [
      { id: 'v2_1', title: 'Washing & Conditioning Fundamentals', videoUrl: 'https://www.youtube.com/embed/dQw4w9WgXcQ?si=RjZ39l22-0mS-1', description: 'Learn the correct way to wash and condition your hair to prevent damage and promote growth.', notes: 'Key takeaways: Use sulfate-free shampoo, focus shampoo on scalp, conditioner on ends. Specifications: Rinse thoroughly with cool water.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
      { id: 'v2_2', title: 'Hair Type & Product Selection', videoUrl: 'https://www.youtube.com/embed/dQw4w9WgXcQ?si=RjZ39l22-0mS-1', description: 'Match your hair type (straight, wavy, curly, coily) with the right products for optimal results.', notes: 'Specifications: Fine hair needs lightweight products; thick hair can handle richer formulas. Avoid excessive heat styling.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
      { id: 'v2_3', title: 'Styling for Your Face Shape', videoUrl: 'https://www.youtube.com/embed/dQw4w9WgXcQ?si=RjZ39l22-0mS-1', description: 'Discover hairstyles that best complement your facial features and enhance your overall look.', notes: 'Specifications: Oval faces are versatile; round faces benefit from volume on top; square faces from softer lines.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
    ]
  },
  {
    id: 'topic3',
    title: 'Styling & Wardrobe',
    description: 'Curate a wardrobe that reflects your personal brand and enhances your physique.',
    videos: [
      { id: 'v3_1', title: 'Understanding Body Proportions', videoUrl: 'https://www.youtube.com/embed/dQw4w9WgXcQ?si=RjZ39l22-0mS-1', description: 'Learn to identify your body shape and how to dress to create balanced proportions.', notes: 'Key takeaways: Focus on creating visual balance. Specifications: Vertical stripes elongate, horizontal stripes widen.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
      { id: 'v3_2', title: 'Color Theory for Clothing', videoUrl: 'https://www.youtube.com/embed/dQw4w9WgXcQ?si=RjZ39l22-0mS-1', description: 'Discover how to choose colors that complement your skin tone and create harmonious outfits.', notes: 'Specifications: Warm skin tones suit earthy colors; cool skin tones suit blues and greens. Use a color wheel.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
      { id: 'v3_3', title: 'Building a Versatile Wardrobe', videoUrl: 'https://www.youtube.com/embed/dQw4w9WgXcQ?si=RjZ39l22-0mS-1', description: 'Identify essential pieces that can be mixed and matched for various occasions, maximizing your style with fewer items.', notes: 'Specifications: Invest in quality basics (white shirt, dark jeans, classic blazer). Consider a capsule wardrobe approach.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
    ]
  },
  {
    id: 'topic4',
    title: 'Fitness & Posture Basics',
    description: 'Lay the groundwork for a healthier physique and confident stance through fundamental fitness and posture exercises.',
    videos: [
      { id: 'v4_1', title: 'Core Strength for Posture', videoUrl: 'https://www.youtube.com/embed/dQw4w9WgXcQ?si=RjZ39l22-0mS-1', description: 'Strengthen your core muscles to improve posture and reduce back pain.', notes: 'Exercises: Planks, bird-dog, dead bug. Specifications: Engage core, keep spine neutral. Perform 3 sets of 30 seconds.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
      { id: 'v4_2', title: 'Basic Full-Body Workout', videoUrl: 'https://www.youtube.com/embed/dQw4w9WgXcQ?si=RjZ39l22-0mS-1', description: 'A beginner-friendly routine to build foundational strength and improve overall fitness.', notes: 'Exercises: Squats, push-ups (modified), lunges, rows (bodyweight). Specifications: Focus on form over quantity. Warm-up and cool-down are crucial.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
      { id: 'v4_3', title: 'Flexibility & Mobility Drills', videoUrl: 'https://www.youtube.com/embed/dQw4w9WgXcQ?si=RjZ39l22-0mS-1', description: 'Enhance your range of motion and prevent stiffness with simple stretching and mobility exercises.', notes: 'Stretches: Hamstring stretch, hip flexor stretch, shoulder rolls. Specifications: Hold stretches for 20-30 seconds. Do not bounce.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
    ]
  },
  {
    id: 'topic5',
    title: 'Confidence & Presence',
    description: 'Cultivate an aura of confidence and improve your overall presence through non-verbal communication and self-perception.',
    videos: [
      { id: 'v5_1', title: 'Mastering Body Language', videoUrl: 'https://www.youtube.com/embed/dQw4w9WgXcQ?si=RjZ39l22-0mS-1', description: 'Learn to project confidence through open posture, eye contact, and purposeful gestures.', notes: 'Key takeaways: Stand tall, shoulders back, chin up. Avoid fidgeting. Specifications: Practice in front of a mirror.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
      { id: 'v5_2', title: 'The Power of Eye Contact', videoUrl: 'https://www.youtube.com/embed/dQw4w9WgXcQ?si=RjZ39l22-0mS-1', description: 'Understand how appropriate eye contact can build rapport and convey sincerity.', notes: 'Specifications: Aim for 60-70% eye contact during conversations. Avoid staring.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
      { id: 'v5_3', title: 'Developing a Confident Voice', videoUrl: 'https://www.youtube.com/embed/dQw4w9WgXcQ?si=RjZ39l22-0mS-1', description: 'Learn techniques to modulate your voice for clarity, authority, and warmth.', notes: 'Specifications: Speak from your diaphragm, avoid uptalk, vary your pitch and pace. Practice reading aloud.', pdfUrl: 'https://www.africau.edu/images/default/sample.pdf' },
    ]
  },
];

// --- Components ---

// Header Component
const Header = ({ currentPage, setCurrentPage, user, handleLogout }) => {
  return (
    <header className="bg-gray-900 text-white p-4 shadow-md">
      <nav className="container mx-auto flex flex-wrap items-center justify-between">
        <div className="text-2xl font-bold text-teal-300 cursor-pointer" onClick={() => setCurrentPage('home')}>
          Look Maxxing Academy
        </div>
        <div className="flex flex-wrap items-center space-x-4 mt-2 md:mt-0">
          <button
            className={`px-3 py-2 rounded-md transition duration-300 ${currentPage === 'home' ? 'bg-teal-700' : 'hover:bg-gray-700'}`}
            onClick={() => setCurrentPage('home')}
          >
            Home
          </button>
          <button
            className={`px-3 py-2 rounded-md transition duration-300 ${currentPage === 'course-overview' ? 'bg-teal-700' : 'hover:bg-gray-700'}`}
            onClick={() => setCurrentPage('course-overview')}
          >
            Course Overview
          </button>
          {user ? (
            <>
              <button
                className={`px-3 py-2 rounded-md transition duration-300 ${currentPage === 'dashboard' ? 'bg-teal-700' : 'hover:bg-gray-700'}`}
                onClick={() => setCurrentPage('dashboard')}
              >
                Dashboard
              </button>
              <button
                className="bg-red-600 hover:bg-red-700 px-4 py-2 rounded-md transition duration-300"
                onClick={handleLogout}
              >
                Logout
              </button>
            </>
          ) : (
            <button
              className={`px-3 py-2 rounded-md transition duration-300 ${currentPage === 'auth' ? 'bg-teal-700' : 'hover:bg-gray-700'}`}
              onClick={() => setCurrentPage('auth')}
            >
              Login / Signup
            </button>
          )}
        </div>
      </nav>
    </header>
  );
};

// Home Page Component
const HomePage = ({ setCurrentPage }) => {
  return (
    <div className="min-h-[calc(100vh-80px)] bg-gradient-to-br from-gray-800 to-gray-900 text-white flex flex-col items-center justify-center p-6 text-center">
      <h1 className="text-5xl md:text-6xl font-extrabold mb-6 text-teal-300 drop-shadow-lg">
        Unlock Your Full Potential
      </h1>
      <p className="text-xl md:text-2xl mb-8 max-w-3xl leading-relaxed">
        Welcome to the **Look Maxxing Academy**, your definitive guide to mastering personal aesthetics, boosting confidence, and revealing your best self.
      </p>
      <button
        className="bg-teal-600 hover:bg-teal-700 text-white font-bold py-3 px-8 rounded-full text-lg shadow-lg transform transition duration-300 hover:scale-105"
        onClick={() => setCurrentPage('course-overview')}
      >
        Explore The Look Maxxing Series
      </button>
      <div className="mt-12 w-full max-w-4xl">
        <h2 className="text-3xl font-bold mb-6 text-teal-200">What You'll Gain</h2>
        <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
          <div className="bg-gray-700 p-6 rounded-lg shadow-xl hover:shadow-2xl transition duration-300 transform hover:-translate-y-1">
            <h3 className="text-xl font-semibold mb-3 text-teal-100">Enhanced Appearance</h3>
            <p className="text-gray-300">Practical, actionable steps for skincare, haircare, and styling.</p>
          </div>
          <div className="bg-gray-700 p-6 rounded-lg shadow-xl hover:shadow-2xl transition duration-300 transform hover:-translate-y-1">
            <h3 className="text-xl font-semibold mb-3 text-teal-100">Boosted Confidence</h3>
            <p className="text-gray-300">Develop a powerful presence and self-assurance from within.</p>
          </div>
          <div className="bg-gray-700 p-6 rounded-lg shadow-xl hover:shadow-2xl transition duration-300 transform hover:-translate-y-1">
            <h3 className="text-xl font-semibold mb-3 text-teal-100">Holistic Transformation</h3>
            <p className="text-gray-300">A comprehensive approach to physical and personal development.</p>
          </div>
        </div>
      </div>
    </div>
  );
};

// Course Overview Component
const CourseOverview = ({ setCurrentPage, setSelectedTopic }) => {
  const { user } = useContext(AuthContext);
  const { completedLessons } = useContext(CourseContext);

  const getTopicProgress = (topicId) => {
    const topic = courseContent.find(t => t.id === topicId);
    if (!topic || !user) return 0;
    const totalLessons = topic.videos.length;
    if (totalLessons === 0) return 0;
    const completedInTopic = topic.videos.filter(video => completedLessons[video.id]).length;
    return Math.round((completedInTopic / totalLessons) * 100);
  };

  return (
    <div className="container mx-auto p-6 bg-gray-900 text-white min-h-[calc(100vh-80px)]">
      <h1 className="text-4xl font-bold mb-8 text-center text-teal-300">The Look Maxxing Series</h1>
      <p className="text-lg text-gray-300 mb-10 text-center max-w-3xl mx-auto">
        Embark on a transformative journey through our comprehensive "Look Maxxing Series." Each topic is meticulously designed to provide you with actionable insights and practical techniques to enhance your appearance and confidence.
      </p>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
        {courseContent.map(topic => (
          <div
            key={topic.id}
            className="bg-gray-800 rounded-lg shadow-xl p-6 cursor-pointer hover:bg-gray-700 transition duration-300 transform hover:-translate-y-1"
            onClick={() => {
              setSelectedTopic(topic.id);
              setCurrentPage('topic-page');
            }}
          >
            <h2 className="text-2xl font-semibold mb-3 text-teal-200">{topic.title}</h2>
            <p className="text-gray-400 mb-4">{topic.description}</p>
            {user && (
              <div className="w-full bg-gray-600 rounded-full h-2.5 mt-4">
                <div
                  className="bg-teal-500 h-2.5 rounded-full transition-all duration-500"
                  style={{ width: `${getTopicProgress(topic.id)}%` }}
                ></div>
                <span className="text-sm text-gray-300 mt-1 block">{getTopicProgress(topic.id)}% Complete</span>
              </div>
            )}
            <button className="mt-4 bg-teal-600 hover:bg-teal-700 text-white py-2 px-4 rounded-md text-sm">
              View Topic
            </button>
          </div>
        ))}
      </div>
    </div>
  );
};

// Topic Page Component
const TopicPage = ({ setCurrentPage, selectedTopic, setSelectedVideo }) => {
  const topic = courseContent.find(t => t.id === selectedTopic);
  const { user } = useContext(AuthContext);
  const { completedLessons } = useContext(CourseContext);

  if (!topic) {
    return <div className="text-white text-center p-8">Topic not found.</div>;
  }

  return (
    <div className="container mx-auto p-6 bg-gray-900 text-white min-h-[calc(100vh-80px)]">
      <h1 className="text-4xl font-bold mb-6 text-teal-300 text-center">{topic.title}</h1>
      <p className="text-lg text-gray-300 mb-8 max-w-3xl mx-auto text-center">{topic.description}</p>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {topic.videos.map(video => (
          <div
            key={video.id}
            className="bg-gray-800 rounded-lg shadow-xl p-5 cursor-pointer hover:bg-gray-700 transition duration-300 transform hover:-translate-y-1"
            onClick={() => {
              setSelectedVideo(video.id);
              setCurrentPage('lesson-page');
            }}
          >
            <h3 className="text-xl font-semibold mb-2 text-teal-200">{video.title}</h3>
            <p className="text-gray-400 text-sm mb-3 line-clamp-2">{video.description}</p>
            {user && completedLessons[video.id] && (
              <span className="text-green-400 text-sm font-medium flex items-center">
                <svg className="w-4 h-4 mr-1" fill="currentColor" viewBox="0 0 20 20">
                  <path fillRule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clipRule="evenodd"></path>
                </svg>
                Completed
              </span>
            )}
            <button className="mt-4 bg-teal-600 hover:bg-teal-700 text-white py-2 px-4 rounded-md text-sm">
              Watch Lesson
            </button>
          </div>
        ))}
      </div>
      <div className="text-center mt-10">
        <button
          className="bg-gray-700 hover:bg-gray-600 text-white py-2 px-6 rounded-md transition duration-300"
          onClick={() => setCurrentPage('course-overview')}
        >
          Back to Course Overview
        </button>
      </div>
    </div>
  );
};

// Lesson Page Component
const LessonPage = ({ setCurrentPage, selectedTopic, selectedVideo }) => {
  const topic = courseContent.find(t => t.id === selectedTopic);
  const video = topic?.videos.find(v => v.id === selectedVideo);
  const { user, userId } = useContext(AuthContext);
  const { completedLessons, setCompletedLessons } = useContext(CourseContext);
  const [showCompletionMessage, setShowCompletionMessage] = useState(false);

  if (!video) {
    return <div className="text-white text-center p-8">Video not found.</div>;
  }

  const markLessonComplete = async () => {
    if (!user || !userId) {
      console.log("User not logged in, cannot mark complete.");
      return;
    }
    try {
      const userProgressRef = doc(db, 'artifacts', appId, 'users', userId, 'progress', 'courseProgress');
      await setDoc(userProgressRef, { [video.id]: true }, { merge: true });
      setCompletedLessons(prev => ({ ...prev, [video.id]: true }));
      setShowCompletionMessage(true);
      setTimeout(() => setShowCompletionMessage(false), 3000); // Hide message after 3 seconds
    } catch (error) {
      console.error("Error marking lesson complete:", error);
    }
  };

  return (
    <div className="container mx-auto p-6 bg-gray-900 text-white min-h-[calc(100vh-80px)]">
      <h1 className="text-4xl font-bold mb-6 text-teal-300 text-center">{video.title}</h1>

      <div className="bg-gray-800 rounded-lg shadow-xl p-4 md:p-6 mb-8">
        <div className="relative w-full" style={{ paddingBottom: '56.25%' /* 16:9 Aspect Ratio */ }}>
          <iframe
            className="absolute top-0 left-0 w-full h-full rounded-md"
            src={video.videoUrl}
            title={video.title}
            frameBorder="0"
            allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
            allowFullScreen
          ></iframe>
        </div>

        <div className="mt-6">
          <h2 className="text-2xl font-semibold mb-3 text-teal-200">Description</h2>
          <p className="text-gray-300 leading-relaxed">{video.description}</p>

          <h2 className="text-2xl font-semibold mt-6 mb-3 text-teal-200">Key Takeaways & Specifications</h2>
          <p className="text-gray-300 leading-relaxed">{video.notes}</p>

          <div className="mt-8 flex flex-wrap gap-4 justify-center">
            {video.pdfUrl && (
              <a
                href={video.pdfUrl}
                target="_blank"
                rel="noopener noreferrer"
                className="bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-6 rounded-md flex items-center transition duration-300 transform hover:scale-105"
              >
                <svg className="w-5 h-5 mr-2" fill="currentColor" viewBox="0 0 20 20">
                  <path fillRule="evenodd" d="M4 4a2 2 0 012-2h4.586A2 2 0 0112 2.414L14.586 5A2 2 0 0115 6.414V16a2 2 0 01-2 2H6a2 2 0 01-2-2V4zm2 2h2v2H6V6zm0 4h2v2H6v-2zm0 4h2v2H6v-2zm4-8h2v2h-2V6zm0 4h2v2h-2v-2zm0 4h2v2h-2v-2z" clipRule="evenodd"></path>
                </svg>
                Download PDF Guide
              </a>
            )}

            {user && !completedLessons[video.id] && (
              <button
                onClick={markLessonComplete}
                className="bg-green-600 hover:bg-green-700 text-white font-bold py-3 px-6 rounded-md flex items-center transition duration-300 transform hover:scale-105"
              >
                <svg className="w-5 h-5 mr-2" fill="currentColor" viewBox="0 0 20 20">
                  <path fillRule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clipRule="evenodd"></path>
                </svg>
                Mark as Complete
              </button>
            )}
            {user && completedLessons[video.id] && (
              <span className="bg-green-700 text-white font-bold py-3 px-6 rounded-md flex items-center opacity-90">
                <svg className="w-5 h-5 mr-2" fill="currentColor" viewBox="0 0 20 20">
                  <path fillRule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clipRule="evenodd"></path>
                </svg>
                Lesson Completed!
              </span>
            )}
          </div>

          {showCompletionMessage && (
            <div className="mt-4 text-center text-green-400 font-semibold">
              Lesson marked as complete!
            </div>
          )}

          <div className="text-center mt-10">
            <button
              className="bg-gray-700 hover:bg-gray-600 text-white py-2 px-6 rounded-md transition duration-300"
              onClick={() => setCurrentPage('topic-page')}
            >
              Back to Topic
            </button>
          </div>
        </div>
      </div>
    </div>
  );
};

// Auth Forms Component (Login/Signup)
const AuthForms = ({ setCurrentPage }) => {
  const [isLogin, setIsLogin] = useState(true);
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');
    setLoading(true);

    try {
      if (isLogin) {
        await signInWithEmailAndPassword(auth, email, password);
      } else {
        await createUserWithEmailAndPassword(auth, email, password);
      }
      setCurrentPage('dashboard'); // Redirect to dashboard on success
    } catch (err) {
      console.error("Auth error:", err.message);
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-[calc(100vh-80px)] flex items-center justify-center bg-gradient-to-br from-gray-800 to-gray-900 p-4">
      <div className="bg-gray-800 p-8 rounded-lg shadow-xl w-full max-w-md border border-gray-700">
        <h2 className="text-3xl font-bold mb-6 text-center text-teal-300">
          {isLogin ? 'Login' : 'Sign Up'}
        </h2>
        {error && <p className="text-red-500 text-center mb-4">{error}</p>}
        <form onSubmit={handleSubmit} className="space-y-5">
          <div>
            <label className="block text-gray-300 text-sm font-bold mb-2" htmlFor="email">
              Email
            </label>
            <input
              type="email"
              id="email"
              className="shadow appearance-none border border-gray-600 rounded w-full py-3 px-4 text-gray-100 leading-tight focus:outline-none focus:ring-2 focus:ring-teal-500 bg-gray-700"
              placeholder="your@email.com"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              required
            />
          </div>
          <div>
            <label className="block text-gray-300 text-sm font-bold mb-2" htmlFor="password">
              Password
            </label>
            <input
              type="password"
              id="password"
              className="shadow appearance-none border border-gray-600 rounded w-full py-3 px-4 text-gray-100 leading-tight focus:outline-none focus:ring-2 focus:ring-teal-500 bg-gray-700"
              placeholder="********"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              required
            />
          </div>
          <button
            type="submit"
            className="bg-teal-600 hover:bg-teal-700 text-white font-bold py-3 px-4 rounded-md w-full transition duration-300 disabled:opacity-50 disabled:cursor-not-allowed"
            disabled={loading}
          >
            {loading ? 'Processing...' : (isLogin ? 'Login' : 'Sign Up')}
          </button>
        </form>
        <p className="text-center text-gray-400 text-sm mt-6">
          {isLogin ? "Don't have an account?" : "Already have an account?"}{' '}
          <button
            onClick={() => setIsLogin(!isLogin)}
            className="text-teal-400 hover:text-teal-300 font-semibold focus:outline-none"
          >
            {isLogin ? 'Sign Up' : 'Login'}
          </button>
        </p>
      </div>
    </div>
  );
};

// Dashboard Component
const Dashboard = () => {
  const { user, userId } = useContext(AuthContext);
  const { completedLessons } = useContext(CourseContext);

  const totalVideos = courseContent.reduce((acc, topic) => acc + topic.videos.length, 0);
  const completedCount = Object.keys(completedLessons).length;
  const overallProgress = totalVideos > 0 ? Math.round((completedCount / totalVideos) * 100) : 0;

  return (
    <div className="container mx-auto p-6 bg-gray-900 text-white min-h-[calc(100vh-80px)]">
      <h1 className="text-4xl font-bold mb-8 text-center text-teal-300">Your Dashboard</h1>
      {user ? (
        <div className="bg-gray-800 p-8 rounded-lg shadow-xl max-w-2xl mx-auto border border-gray-700">
          <p className="text-lg mb-4 text-gray-300">
            Welcome, <span className="font-semibold text-teal-200">{user.email || 'Guest User'}</span>!
          </p>
          <p className="text-sm text-gray-400 mb-6">
            Your User ID: <span className="font-mono text-teal-400 break-all">{userId}</span>
          </p>

          <h2 className="text-2xl font-semibold mb-4 text-teal-200">Course Progress</h2>
          <div className="mb-4">
            <p className="text-gray-300 mb-2">Overall Progress: {overallProgress}%</p>
            <div className="w-full bg-gray-700 rounded-full h-3">
              <div
                className="bg-teal-500 h-3 rounded-full transition-all duration-500"
                style={{ width: `${overallProgress}%` }}
              ></div>
            </div>
            <p className="text-gray-400 text-sm mt-1">{completedCount} of {totalVideos} lessons completed.</p>
          </div>

          <h3 className="text-xl font-semibold mb-3 text-teal-200">Completed Lessons</h3>
          {completedCount > 0 ? (
            <ul className="list-disc list-inside text-gray-300 space-y-2">
              {courseContent.map(topic => (
                topic.videos.map(video => (
                  completedLessons[video.id] && (
                    <li key={video.id}>
                      <span className="font-medium text-teal-400">{topic.title}:</span> {video.title}
                    </li>
                  )
                ))
              ))}
            </ul>
          ) : (
            <p className="text-gray-400">No lessons completed yet. Start learning!</p>
          )}
        </div>
      ) : (
        <p className="text-center text-gray-400 text-lg">
          Please login or sign up to view your dashboard and track progress.
        </p>
      )}
    </div>
  );
};

// Main App Component
const App = () => {
  const [currentPage, setCurrentPage] = useState('home');
  const [selectedTopic, setSelectedTopic] = useState(null);
  const [selectedVideo, setSelectedVideo] = useState(null);
  const [user, setUser] = useState(null);
  const [userId, setUserId] = useState(null);
  const [loadingAuth, setLoadingAuth] = useState(true);
  const [completedLessons, setCompletedLessons] = useState({});

  // Firebase Auth Listener
  useEffect(() => {
    const initializeAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (error) {
        console.error("Firebase initial auth error:", error);
      } finally {
        setLoadingAuth(false);
      }
    };

    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
      if (currentUser) {
        setUserId(currentUser.uid);
        // Listen for user progress
        const userProgressRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'progress', 'courseProgress');
        const unsubscribeProgress = onSnapshot(userProgressRef, (docSnap) => {
          if (docSnap.exists()) {
            setCompletedLessons(docSnap.data());
          } else {
            setCompletedLessons({});
          }
        }, (error) => {
          console.error("Error fetching user progress:", error);
        });
        return () => unsubscribeProgress(); // Cleanup progress listener
      } else {
        setUserId(null);
        setCompletedLessons({});
      }
      setLoadingAuth(false);
    });

    initializeAuth(); // Call initial auth setup
    return () => unsubscribe(); // Cleanup auth listener
  }, []); // Run only once on component mount

  const handleLogout = async () => {
    try {
      await signOut(auth);
      setCurrentPage('home'); // Redirect to home after logout
    } catch (error) {
      console.error("Error logging out:", error);
    }
  };

  if (loadingAuth) {
    return (
      <div className="flex items-center justify-center min-h-screen bg-gray-900 text-teal-300 text-2xl">
        Loading authentication...
      </div>
    );
  }

  return (
    <AuthContext.Provider value={{ user, userId, auth }}>
      <CourseContext.Provider value={{ completedLessons, setCompletedLessons, courseContent }}>
        <div className="min-h-screen bg-gray-950 font-sans text-gray-100">
          <style>
            {`
            @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
            body { font-family: 'Inter', sans-serif; }
            `}
          </style>
          <Header currentPage={currentPage} setCurrentPage={setCurrentPage} user={user} handleLogout={handleLogout} />
          <main>
            {currentPage === 'home' && <HomePage setCurrentPage={setCurrentPage} />}
            {currentPage === 'course-overview' && (
              <CourseOverview setCurrentPage={setCurrentPage} setSelectedTopic={setSelectedTopic} />
            )}
            {currentPage === 'topic-page' && selectedTopic && (
              <TopicPage setCurrentPage={setCurrentPage} selectedTopic={selectedTopic} setSelectedVideo={setSelectedVideo} />
            )}
            {currentPage === 'lesson-page' && selectedTopic && selectedVideo && (
              <LessonPage setCurrentPage={setCurrentPage} selectedTopic={selectedTopic} selectedVideo={selectedVideo} />
            )}
            {currentPage === 'auth' && <AuthForms setCurrentPage={setCurrentPage} />}
            {currentPage === 'dashboard' && <Dashboard />}
          </main>
        </div>
      </CourseContext.Provider>
    </AuthContext.Provider>
  );
};

export default App;
