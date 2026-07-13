/**
 * @license
 * SPDX-License-Identifier: Apache-2.0
 */

import React, { useState, useEffect, useCallback } from 'react';
import { Article, NewsCategory, NewsUiState, RetrofitLog, FcmNotification, SimulatorScreen } from './types';
import { AndroidPhoneFrame } from './components/AndroidPhoneFrame';
import { HomeScreen } from './components/screens/HomeScreen';
import { DetailScreen } from './components/screens/DetailScreen';
import { BookmarksScreen } from './components/screens/BookmarksScreen';
import { SearchScreen } from './components/screens/SearchScreen';
import { NotificationsScreen } from './components/screens/NotificationsScreen';
import { FcmPushLab } from './components/workbench/FcmPushLab';
import { ArchitectureInspector } from './components/workbench/ArchitectureInspector';
import { SplashScreen } from './components/screens/SplashScreen';
import { fetchBloggerArticles, BLOGGER_JSON_FEED_URL } from './services/bloggerService';
import {
  isNativeCapacitor,
  initCapacitorNativeUI,
  initCapacitorNetworkListener,
  initCapacitorPushNotifications,
  triggerHapticLight,
  triggerHapticMedium,
  saveNativeBookmarks,
  loadNativeBookmarks,
  saveNativeArticlesCache,
  loadNativeArticlesCache
} from './services/capacitorService';
import { Flame, Bell, Database, Smartphone, Code, Wifi, Sparkles, Download } from 'lucide-react';

const INITIAL_ARTICLES: Article[] = [
  {
    id: 'art-1',
    title: 'Google DeepMind Unveils Gemini 3.5: Next-Gen Autonomous Reasoning Engine',
    summary: 'The new architecture introduces native multimodal tool chaining and real-time reflection loops, outperforming human experts in complex engineering benchmarks.',
    content: 'In a landmark keynote today, researchers at Google DeepMind revealed the Gemini 3.5 series. The model introduces massive upgrades in real-time latency, native speech synthesis, and autonomous task execution. Early enterprise testers report a 4x increase in software development productivity and automated bug resolution. The new architecture also features enhanced safety guardrails and verifiable citation grounding.',
    author: 'Elena Rostova',
    sourceName: 'TechCrunch 24',
    publishedAt: '10 mins ago',
    imageUrl: 'https://images.unsplash.com/photo-1618005182384-a83a8bd57fbe?w=800&auto=format&fit=crop&q=80',
    category: 'AI',
    url: 'https://flashnews24.io/articles/gemini-3-5-unveiled',
    readTimeMinutes: 4,
    isBreaking: true,
    sentiment: 'Urgent'
  },
  {
    id: 'art-2',
    title: 'Global Markets Rally as Green Energy Adoption Hits Tipping Point in Europe',
    summary: 'Solar and wind energy output officially surpassed fossil fuels across EU grids during Q2, triggering massive institutional investments in clean tech.',
    content: 'Stock markets across Europe and North America surged today following data from the International Energy Agency showing that renewable sources provided over 52% of total electricity generation last quarter. Solar grid installations grew by 38% year-over-year, driven by breakthrough perovskite panel efficiency and grid-scale battery storage cost drops.',
    author: 'Marcus Vance',
    sourceName: 'Bloomberg News',
    publishedAt: '32 mins ago',
    imageUrl: 'https://images.unsplash.com/photo-1466611653911-95081537e5b7?w=800&auto=format&fit=crop&q=80',
    category: 'Business',
    url: 'https://flashnews24.io/articles/green-energy-tipping-point',
    readTimeMinutes: 5,
    isBreaking: false,
    sentiment: 'Positive'
  },
  {
    id: 'art-3',
    title: 'James Webb Telescope Spots Earliest Water-Rich Atmosphere on Exoplanet K2-18c',
    summary: 'Spectroscopic analysis reveals clear signatures of water vapor and carbon-bearing molecules in the habitable zone of a nearby red dwarf star.',
    content: 'Astronomers using the James Webb Space Telescope have detected definitive signatures of water vapor, methane, and carbon dioxide in the atmosphere of exoplanet K2-18c, located 120 light-years from Earth. The findings suggest an ocean-covered sub-Neptune world capable of sustaining liquid water on its surface.',
    author: 'Dr. Aris Thorne',
    sourceName: 'Scientific American',
    publishedAt: '1 hour ago',
    imageUrl: 'https://images.unsplash.com/photo-1451187580459-43490279c0fa?w=800&auto=format&fit=crop&q=80',
    category: 'Science',
    url: 'https://flashnews24.io/articles/jwst-water-atmosphere',
    readTimeMinutes: 6,
    isBreaking: false,
    sentiment: 'Analytical'
  },
  {
    id: 'art-4',
    title: 'Android 16 Developer Preview Brings Real-time Spatial Audio & Satellite SOS',
    summary: 'Google releases the first beta build featuring Jetpack Compose 1.8 optimizations, dynamic material color synthesis, and satellite emergency messaging.',
    content: 'The Android development ecosystem is buzzing following the release of the Android 16 Developer Preview. Highlights include native spatial audio rendering with dynamic head tracking, automated background battery management powered by on-device AI, and built-in satellite SOS APIs for developers building mission-critical field apps.',
    author: 'Sarah Jenkins',
    sourceName: 'Android Central',
    publishedAt: '2 hours ago',
    imageUrl: 'https://images.unsplash.com/photo-1607252650355-f7fd0460ccdb?w=800&auto=format&fit=crop&q=80',
    category: 'Tech',
    url: 'https://flashnews24.io/articles/android-16-preview',
    readTimeMinutes: 3,
    isBreaking: false,
    sentiment: 'Positive'
  },
  {
    id: 'art-5',
    title: 'Championship Thriller: Underdog FC Tokyo Secures Last-Minute Victory in Extra Time',
    summary: 'A dramatic 94th-minute volley seals a historic 3-2 comeback against the reigning champions in front of a sold-out stadium of 65,000 fans.',
    content: 'In one of the most memorable matches of the decade, FC Tokyo overturned a 2-0 halftime deficit to defeat the league champions 3-2. Teenager Kenji Sato scored the winner in the 94th minute with a stunning volley from 25 yards out, igniting euphoric celebrations across the stadium.',
    author: 'David Beckham Jr.',
    sourceName: 'ESPN Global',
    publishedAt: '3 hours ago',
    imageUrl: 'https://images.unsplash.com/photo-1508098682722-e99c43a406b2?w=800&auto=format&fit=crop&q=80',
    category: 'Sports',
    url: 'https://flashnews24.io/articles/fc-tokyo-thriller',
    readTimeMinutes: 4,
    isBreaking: false,
    sentiment: 'Positive'
  },
  {
    id: 'art-6',
    title: 'Next-Gen Quantum Chip Achieves 99.9% Error-Correction Gate Fidelity',
    summary: 'Physicists demonstrate fault-tolerant topological qubits operating at room temperature, paving the way for commercial quantum supercomputers by 2028.',
    content: 'A consortium of quantum researchers has announced a major breakthrough in qubit stability. By utilizing topological braiding in diamond nitrogen-vacancy centers, the team achieved a 99.9% two-qubit gate fidelity without requiring liquid helium dilution refrigerators. This milestone removes the biggest hurdle to commercial quantum scaling.',
    author: 'Prof. Chen Wei',
    sourceName: 'Nature Technology',
    publishedAt: '5 hours ago',
    imageUrl: 'https://images.unsplash.com/photo-1635070041078-e363dbe005cb?w=800&auto=format&fit=crop&q=80',
    category: 'Tech',
    url: 'https://flashnews24.io/articles/quantum-chip-fidelity',
    readTimeMinutes: 5,
    isBreaking: false,
    sentiment: 'Analytical'
  }
];

export default function App() {
  const [articles, setArticles] = useState<Article[]>(INITIAL_ARTICLES);
  const [bookmarkedIds, setBookmarkedIds] = useState<string[]>(['art-1', 'art-4']);
  const [selectedCategory, setSelectedCategory] = useState<NewsCategory>('All');
  const [activeScreen, setActiveScreen] = useState<SimulatorScreen>('home');
  const [selectedArticle, setSelectedArticle] = useState<Article | null>(null);
  const [isDarkMode, setIsDarkMode] = useState(true);
  const [isOffline, setIsOffline] = useState(false);
  const [isRefreshing, setIsRefreshing] = useState(false);
  const [isGeneratingAiNews, setIsGeneratingAiNews] = useState(false);
  const [showSplash, setShowSplash] = useState(true);
  const [isSyncingBlogger, setIsSyncingBlogger] = useState(false);

  // Workbench tabs for responsive screens
  const [mobileWorkbenchTab, setMobileWorkbenchTab] = useState<'phone' | 'fcm' | 'inspector'>('phone');

  // FCM Notifications
  const [notifications, setNotifications] = useState<FcmNotification[]>([
    {
      id: 'fcm-init-1',
      title: '🚨 FLASH BREAKING: Gemini 3.5 Unveiled',
      body: 'Google DeepMind announces autonomous reasoning architecture with real-time reflection.',
      priority: 'HIGH',
      timestamp: '10 mins ago',
      articleId: 'art-1'
    }
  ]);
  const [activeBanner, setActiveBanner] = useState<FcmNotification | null>(null);

  // Retrofit HTTP Interceptor Logs
  const [retrofitLogs, setRetrofitLogs] = useState<RetrofitLog[]>([
    {
      id: 'log-1',
      timestamp: new Date().toLocaleTimeString(),
      method: 'GET',
      url: BLOGGER_JSON_FEED_URL,
      status: 200,
      durationMs: 142,
      responseSize: '42.8 KB'
    }
  ]);

  const addRetrofitLog = useCallback((method: 'GET' | 'POST', url: string, status: number, duration: number, size: string) => {
    const newLog: RetrofitLog = {
      id: `log-${Date.now()}-${Math.random().toString(36).substr(2, 4)}`,
      timestamp: new Date().toLocaleTimeString(),
      method,
      url,
      status,
      durationMs: duration,
      responseSize: size
    };
    setRetrofitLogs(prev => [newLog, ...prev]);
  }, []);

  const handleRefreshNews = useCallback(() => {
    setIsRefreshing(true);
    triggerHapticMedium();
    const startTime = Date.now();
    
    if (isOffline) {
      setTimeout(() => {
        setIsRefreshing(false);
        addRetrofitLog('GET', BLOGGER_JSON_FEED_URL, 503, Date.now() - startTime, '0 B');
      }, 500);
      return;
    }

    fetchBloggerArticles('All')
      .then(liveArticles => {
        if (liveArticles && liveArticles.length > 0) {
          setArticles(prev => {
            const aiStories = prev.filter(a => a.id.startsWith('art-ai-') || a.id.startsWith('art-live-'));
            const merged = [...aiStories, ...liveArticles];
            const unique = Array.from(new Map(merged.map(item => [item.id, item])).values());
            return unique;
          });
          addRetrofitLog('GET', `${BLOGGER_JSON_FEED_URL}?category=all`, 200, Date.now() - startTime, '48.2 KB');
        } else {
          addRetrofitLog('GET', `${BLOGGER_JSON_FEED_URL}?category=all`, 304, Date.now() - startTime, '12.4 KB');
        }
      })
      .catch(err => {
        addRetrofitLog('GET', BLOGGER_JSON_FEED_URL, 500, Date.now() - startTime, '0 B');
      })
      .finally(() => {
        setIsRefreshing(false);
      });
  }, [isOffline, addRetrofitLog]);

  useEffect(() => {
    handleRefreshNews();
    initCapacitorNativeUI();

    const removeNetListenerPromise = initCapacitorNetworkListener((connected) => {
      setIsOffline(!connected);
    });

    initCapacitorPushNotifications((title, body, articleId) => {
      handleBroadcastNotification(title, body, 'HIGH', articleId);
    });

    loadNativeBookmarks().then(saved => {
      if (saved && saved.length > 0) {
        setBookmarkedIds(saved);
      }
    });

    loadNativeArticlesCache().then(cached => {
      if (cached && cached.length > 0) {
        setArticles(cached);
      }
    });

    const interval = setInterval(() => {
      if (!isOffline) {
        fetchBloggerArticles('All').then(liveArticles => {
          if (liveArticles && liveArticles.length > 0) {
            setArticles(prev => {
              const existingIds = new Set(prev.map(a => a.id));
              const newArrivals = liveArticles.filter(a => !existingIds.has(a.id));
              if (newArrivals.length > 0) {
                const newest = newArrivals[0];
                handleBroadcastNotification(
                  `🚨 NEW BLOGGER POST (${newest.category}): ${newest.title.slice(0, 45)}...`,
                  newest.summary || 'Real-time feed sync received via Firebase Cloud Messaging.',
                  'HIGH',
                  newest.id
                );
              }
              const aiStories = prev.filter(a => a.id.startsWith('art-ai-') || a.id.startsWith('art-live-'));
              const merged = [...aiStories, ...liveArticles];
              const unique = Array.from(new Map(merged.map(item => [item.id, item])).values());
              return unique;
            });
          }
        }).catch(() => {});
      }
    }, 45000);
    return () => {
      clearInterval(interval);
      removeNetListenerPromise.then(remove => remove());
    };
  }, [handleRefreshNews, isOffline]);

  useEffect(() => {
    saveNativeBookmarks(bookmarkedIds);
  }, [bookmarkedIds]);

  useEffect(() => {
    if (articles.length > 0) {
      saveNativeArticlesCache(articles);
    }
  }, [articles]);

  const handleToggleBookmark = (id: string) => {
    triggerHapticLight();
    setBookmarkedIds(prev => 
      prev.includes(id) ? prev.filter(item => item !== id) : [...prev, id]
    );
  };

  const handleSelectArticle = (article: Article) => {
    triggerHapticLight();
    setSelectedArticle(article);
    setActiveScreen('detail');
  };

  const handleShareArticle = (article: Article) => {
    if (navigator.share) {
      navigator.share({
        title: article.title,
        text: article.summary,
        url: article.url
      }).catch(() => {});
    } else {
      navigator.clipboard.writeText(`${article.title}\n${article.url}`);
      alert(`Shared: ${article.title}\nLink copied to clipboard!`);
    }
  };

  const handleBroadcastNotification = (title: string, body: string, priority: 'HIGH' | 'NORMAL', articleId?: string) => {
    const newNotif: FcmNotification = {
      id: `fcm-${Date.now()}`,
      title,
      body,
      priority,
      timestamp: 'Just now',
      articleId
    };
    setNotifications(prev => [newNotif, ...prev]);
    setActiveBanner(newNotif);
    
    // Auto hide banner after 6 seconds
    setTimeout(() => {
      setActiveBanner(curr => curr?.id === newNotif.id ? null : curr);
    }, 6000);
  };

  const handleGenerateAiBreakingArticle = async (topic: string) => {
    setIsGeneratingAiNews(true);
    const startTime = Date.now();
    try {
      const res = await fetch('/api/news/ai-breaking', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ topic })
      });
      const data = await res.json();
      if (data.article) {
        setArticles(prev => [data.article, ...prev]);
        addRetrofitLog('POST', 'https://api.flashnews24.io/top-headlines/ai-dispatch', 201, Date.now() - startTime, '4.2 KB');
        
        // Broadcast push alert for this new story
        handleBroadcastNotification(
          `⚡ BREAKING (${data.article.category}): ${data.article.title.slice(0, 45)}...`,
          data.article.summary,
          'HIGH',
          data.article.id
        );
      }
    } catch (err) {
      addRetrofitLog('POST', 'https://api.flashnews24.io/top-headlines/ai-dispatch', 500, Date.now() - startTime, '0 B');
    } finally {
      setIsGeneratingAiNews(false);
    }
  };

  const handleSyncLatestBloggerPostPush = async () => {
    setIsSyncingBlogger(true);
    const startTime = Date.now();
    try {
      const liveArticles = await fetchBloggerArticles('All');
      if (liveArticles && liveArticles.length > 0) {
        const latest = liveArticles[0];
        setArticles(prev => {
          const aiStories = prev.filter(a => a.id.startsWith('art-ai-') || a.id.startsWith('art-live-'));
          const merged = [...aiStories, ...liveArticles];
          return Array.from(new Map(merged.map(item => [item.id, item])).values());
        });
        addRetrofitLog('GET', `${BLOGGER_JSON_FEED_URL}?category=all&fcm_trigger=1`, 200, Date.now() - startTime, '48.2 KB');
        
        handleBroadcastNotification(
          `🚨 NEW BLOGGER POST (${latest.category}): ${latest.title.slice(0, 45)}...`,
          latest.summary || 'Real-time Blogger headline synced via Retrofit and Room.',
          'HIGH',
          latest.id
        );
      }
    } catch (err) {
      addRetrofitLog('GET', BLOGGER_JSON_FEED_URL, 500, Date.now() - startTime, '0 B');
    } finally {
      setIsSyncingBlogger(false);
    }
  };

  const bookmarkedArticles = articles.filter(a => bookmarkedIds.includes(a.id));

  const uiState: NewsUiState = {
    status: isRefreshing ? 'loading' : 'success',
    articles,
    bookmarkedIds,
    selectedCategory,
    searchQuery: '',
    isRefreshing,
    isOfflineMode: isOffline,
    lastUpdated: new Date().toLocaleTimeString()
  };

  return (
    <div className="flex flex-col h-screen w-screen bg-[#0F1115] text-[#E1E4E8] font-sans overflow-hidden select-none">
      
      {/* Top Header Bar */}
      <header className="h-16 px-8 bg-[#14171C] border-b border-[#2D333B] flex items-center justify-between flex-shrink-0 z-40">
        <div className="flex items-center gap-3">
          <div className="w-10 h-10 rounded-xl bg-gradient-to-tr from-blue-600 to-indigo-500 flex items-center justify-center text-white shadow-lg shadow-blue-900/20">
            <Flame className="w-6 h-6 fill-current animate-pulse" />
          </div>
          <div>
            <div className="flex items-center gap-2">
              <h1 className="text-lg font-bold tracking-tight text-white">
                FlashNews<span className="text-blue-500">24</span>
              </h1>
              <span className="px-2.5 py-0.5 rounded-md bg-blue-600/20 text-blue-400 text-[10px] font-bold uppercase tracking-widest border border-blue-500/30">
                NATIVE ANDROID WORKBENCH
              </span>
            </div>
            <p className="text-xs text-[#E1E4E8] opacity-60 font-medium">
              Kotlin • Jetpack Compose • MVVM • Room Offline SQLite • Retrofit • Firebase Cloud Messaging
            </p>
          </div>
        </div>

        {/* Responsive Mobile Tab Bar Switcher for narrower screens */}
        <div className="flex lg:hidden items-center gap-1 bg-[#1A1D23] p-1 rounded-2xl border border-[#2D333B]">
          <button
            onClick={() => setMobileWorkbenchTab('fcm')}
            className={`px-3 py-1.5 rounded-xl text-xs font-bold transition-all ${
              mobileWorkbenchTab === 'fcm' ? 'bg-blue-600 text-white shadow-md' : 'text-[#E1E4E8] opacity-60 hover:opacity-100'
            }`}
          >
            🚨 FCM Lab
          </button>
          <button
            onClick={() => setMobileWorkbenchTab('phone')}
            className={`px-3 py-1.5 rounded-xl text-xs font-bold transition-all ${
              mobileWorkbenchTab === 'phone' ? 'bg-blue-600 text-white shadow-md' : 'text-[#E1E4E8] opacity-60 hover:opacity-100'
            }`}
          >
            📱 Phone UI
          </button>
          <button
            onClick={() => setMobileWorkbenchTab('inspector')}
            className={`px-3 py-1.5 rounded-xl text-xs font-bold transition-all ${
              mobileWorkbenchTab === 'inspector' ? 'bg-blue-600 text-white shadow-md' : 'text-[#E1E4E8] opacity-60 hover:opacity-100'
            }`}
          >
            🏛️ Inspector
          </button>
        </div>

        {/* Desktop Header Stats */}
        <div className="hidden md:flex items-center gap-3">
          <div className="flex items-center gap-1.5 px-3 py-1.5 rounded-full bg-blue-500/10 border border-blue-500/30 text-xs shadow-sm">
            <span className="w-2 h-2 rounded-full bg-blue-400 animate-pulse"></span>
            <span className="text-blue-300 font-bold">Capacitor 8: {isNativeCapacitor() ? 'NATIVE ANDROID APK' : 'WEB BRIDGE ACTIVE'}</span>
          </div>
          <div className="flex items-center gap-2 px-3.5 py-1.5 rounded-full bg-[#1A1D23] border border-[#2D333B] text-xs shadow-sm">
            <div className="w-2 h-2 rounded-full bg-emerald-500 animate-ping"></div>
            <span className="text-[#E1E4E8] opacity-80 font-medium">Room DB: <strong className="text-white">{bookmarkedIds.length} Bookmarks Cached</strong></span>
          </div>
          <div className="flex items-center gap-2 px-3.5 py-1.5 rounded-full bg-[#1A1D23] border border-[#2D333B] text-xs shadow-sm">
            <span className="text-[#E1E4E8] opacity-80 font-medium">FCM Receiver: <strong className="text-blue-400">ONLINE</strong></span>
          </div>
        </div>
      </header>

      {/* Main 3-Column Studio Layout */}
      <div className="flex-1 flex overflow-hidden p-6 gap-8">
        
        {/* LEFT COLUMN: FCM Push Lab & Live AI News Dispatcher (Hidden on mobile if tab != fcm) */}
        <div className={`w-full lg:w-[360px] xl:w-[400px] flex-shrink-0 h-full ${
          mobileWorkbenchTab === 'fcm' ? 'flex' : 'hidden lg:flex'
        }`}>
          <FcmPushLab
            onBroadcastNotification={handleBroadcastNotification}
            onGenerateAiBreakingArticle={handleGenerateAiBreakingArticle}
            isGeneratingAiNews={isGeneratingAiNews}
            onSyncBloggerPostPush={handleSyncLatestBloggerPostPush}
            isSyncingBlogger={isSyncingBlogger}
          />
        </div>

        {/* CENTER COLUMN: Android Native Smartphone Simulator (Pixel Frame) */}
        <div className={`flex-1 flex flex-col items-center justify-center min-w-[380px] h-full ${
          mobileWorkbenchTab === 'phone' ? 'flex' : 'hidden lg:flex'
        }`}>
          <div className="flex items-center gap-4 mb-3">
            {/* Phone Bottom/Top Navigation Bar Selector */}
            <div className="flex items-center gap-1.5 bg-[#1A1D23] p-1.5 rounded-2xl border border-[#2D333B] shadow-xl">
              <button
                onClick={() => setActiveScreen('home')}
                className={`px-4 py-2 rounded-xl text-xs font-bold transition-all flex items-center gap-1.5 ${
                  activeScreen === 'home' || activeScreen === 'detail'
                    ? 'bg-blue-600 text-white shadow-md shadow-blue-900/20'
                    : 'text-[#E1E4E8] opacity-60 hover:opacity-100'
                }`}
              >
                <Smartphone className="w-3.5 h-3.5" />
                <span>Home Feed</span>
              </button>
              <button
                onClick={() => setActiveScreen('bookmarks')}
                className={`px-4 py-2 rounded-xl text-xs font-bold transition-all flex items-center gap-1.5 ${
                  activeScreen === 'bookmarks'
                    ? 'bg-blue-600 text-white shadow-md shadow-blue-900/20'
                    : 'text-[#E1E4E8] opacity-60 hover:opacity-100'
                }`}
              >
                <Database className="w-3.5 h-3.5" />
                <span>Room Bookmarks ({bookmarkedIds.length})</span>
              </button>
              <button
                onClick={() => setActiveScreen('notifications')}
                className={`px-4 py-2 rounded-xl text-xs font-bold transition-all flex items-center gap-1.5 ${
                  activeScreen === 'notifications'
                    ? 'bg-blue-600 text-white shadow-md shadow-blue-900/20'
                    : 'text-[#E1E4E8] opacity-60 hover:opacity-100'
                }`}
              >
                <Bell className="w-3.5 h-3.5" />
                <span>FCM Alerts ({notifications.length})</span>
              </button>
              <button
                onClick={() => setShowSplash(true)}
                className="px-3 py-2 rounded-xl text-xs font-black bg-gradient-to-r from-blue-600/30 to-indigo-600/30 text-blue-300 hover:text-white border border-blue-500/40 hover:border-blue-400 transition-all flex items-center gap-1 shadow-sm"
                title="Replay Android Native Boot Splash Screen"
              >
                <Sparkles className="w-3.5 h-3.5 animate-spin" />
                <span>⚡ Boot Splash</span>
              </button>
            </div>
          </div>

          <AndroidPhoneFrame
            isDarkMode={isDarkMode}
            onToggleDarkMode={() => setIsDarkMode(!isDarkMode)}
            activeNotification={activeBanner}
            onDismissNotification={() => setActiveBanner(null)}
            onNotificationClick={(notif) => {
              setActiveBanner(null);
              if (notif.articleId) {
                const target = articles.find(a => a.id === notif.articleId);
                if (target) {
                  setSelectedArticle(target);
                  setActiveScreen('detail');
                  return;
                }
              }
              setActiveScreen('notifications');
            }}
            isOffline={isOffline}
            onToggleOffline={() => setIsOffline(!isOffline)}
          >
            {/* Render Boot Splash Screen or active native mobile screen */}
            {showSplash ? (
              <SplashScreen onFinish={() => setShowSplash(false)} />
            ) : (
              <>
                {activeScreen === 'home' && (
              <HomeScreen
                articles={articles}
                bookmarkedIds={bookmarkedIds}
                selectedCategory={selectedCategory}
                onSelectCategory={setSelectedCategory}
                onSelectArticle={handleSelectArticle}
                onToggleBookmark={handleToggleBookmark}
                onShareArticle={handleShareArticle}
                onRefresh={handleRefreshNews}
                onOpenSearch={() => setActiveScreen('search')}
                isRefreshing={isRefreshing}
                isOffline={isOffline}
              />
            )}

            {activeScreen === 'detail' && selectedArticle && (
              <DetailScreen
                article={selectedArticle}
                isBookmarked={bookmarkedIds.includes(selectedArticle.id)}
                onBack={() => setActiveScreen('home')}
                onToggleBookmark={handleToggleBookmark}
                onShare={handleShareArticle}
              />
            )}

            {activeScreen === 'bookmarks' && (
              <BookmarksScreen
                articles={bookmarkedArticles}
                onSelectArticle={handleSelectArticle}
                onRemoveBookmark={handleToggleBookmark}
                onClearAllBookmarks={() => setBookmarkedIds([])}
              />
            )}

            {activeScreen === 'search' && (
              <SearchScreen
                allArticles={articles}
                onSelectArticle={handleSelectArticle}
                onClose={() => setActiveScreen('home')}
              />
            )}

            {activeScreen === 'notifications' && (
              <NotificationsScreen
                notifications={notifications}
                allArticles={articles}
                onSelectArticle={handleSelectArticle}
                onClearNotifications={() => setNotifications([])}
              />
            )}
              </>
            )}
          </AndroidPhoneFrame>
        </div>

        {/* RIGHT COLUMN: MVVM Architecture Inspector & Play Store ZIP Exporter (Hidden on mobile if tab != inspector) */}
        <div className={`w-full lg:w-[420px] xl:w-[480px] flex-shrink-0 h-full ${
          mobileWorkbenchTab === 'inspector' ? 'flex' : 'hidden lg:flex'
        }`}>
          <ArchitectureInspector
            articles={articles}
            bookmarkedIds={bookmarkedIds}
            uiState={uiState}
            retrofitLogs={retrofitLogs}
            isOffline={isOffline}
            onToggleOffline={() => setIsOffline(!isOffline)}
            onClearRoomCache={() => setBookmarkedIds([])}
          />
        </div>
      </div>
    </div>
  );
}

