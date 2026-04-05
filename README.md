===============================================================================
LTX-2 ANIME AGENT - IMPLEMENTATION PLAN
===============================================================================
Version: 2.0.0
Features: AI Transcript Generation, Voice Consistency, Audio Integration
Optimized for: NVIDIA GTX 1660 Ti (6GB VRAM)
===============================================================================

PHASE 1: PROJECT SETUP & FOUNDATION
===============================================================================

STEP 1: Initialize Vite React TypeScript Project
-------------------------------------------
Commands:
npm create vite@latest ltx-anime-agent -- --template react-ts
cd ltx-anime-agent
npm install

STEP 2: Install Tailwind CSS and Dependencies
-------------------------------------------
Packages:
npm install -D tailwindcss postcss autoprefixer @tailwindcss/typography
npx tailwindcss init -p

Create: tailwind.config.js
Content:
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [
    require('@tailwindcss/typography'),
  ],
}

Create: src/index.css
Content:
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --anime-pink: #ff6b9d;
  --anime-blue: #4a90e2;
  --anime-purple: #9b59b6;
  --anime-gold: #f1c40f;
}

body {
  @apply bg-gray-950 text-white;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  overflow-x: hidden;
}

.anime-gradient {
  background: linear-gradient(45deg, var(--anime-pink), var(--anime-blue), var(--anime-purple));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  animation: gradient-shift 3s ease-in-out infinite;
}

@keyframes gradient-shift {
  0%, 100% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
}

.anime-card {
  @apply bg-gray-800 border border-gray-700 rounded-lg transition-all duration-300;
  position: relative;
  overflow: hidden;
}

.anime-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 107, 157, 0.1), transparent);
  transition: left 0.5s;
}

.anime-card:hover::before {
  left: 100%;
}

.anime-card:hover {
  @apply border-pink-500 transform -translate-y-1 shadow-lg shadow-pink-500/20;
}

STEP 3: Create Project Directory Structure
-------------------------------------------
Directories:
src/
├── components/
│   ├── layout/
│   ├── projects/
│   ├── characters/
│   ├── episodes/
│   ├── transcripts/
│   ├── generation/
│   └── settings/
├── hooks/
├── utils/
├── types/
└── assets/

STEP 4: Install Additional Dependencies
-------------------------------------------
Packages:
npm install lucide-react clsx tailwind-merge
npm install -D @types/node

STEP 5: Update Basic Configuration
-------------------------------------------
Edit: vite.config.ts
Content:
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})

Edit: tsconfig.json
Add to compilerOptions:
"baseUrl": ".",
"paths": {
  "@/*": ["src/*"]
}

Edit: index.html
Change title to: <title>LTX-2 Anime Agent 🎬</title>

===============================================================================

PHASE 2: TYPE DEFINITIONS & CORE DATA
===============================================================================

STEP 6: Create TypeScript Interfaces
-------------------------------------------
Create: src/types/index.ts
Content:
// Core Project Types
export interface Project {
  id: string;
  name: string;
  description: string;
  style: AnimationStyle;
  created: string;
  modified: string;
}

export interface Character {
  id: string;
  projectId: string;
  name: string;
  appearance: string;
  personality: string;
  role: string;
  consistencyPrompt: string;
  voiceSettings: VoiceSettings;
  created: string;
}

export interface VoiceSettings {
  voiceId: string;
  pitch: number;
  rate: number;
  volume: number;
  style?: string;
}

export interface Episode {
  id: string;
  projectId: string;
  title: string;
  description: string;
  scenes: Scene[];
  created: string;
}

export interface Scene {
  id: string;
  episodeId: string;
  title: string;
  description: string;
  camera: CameraMovement;
  mood: Mood;
  duration: number;
  characters: string[];
  style?: string;
  transcript?: Transcript;
  generatedVideoUrl?: string;
  generatedAudioUrl?: string;
  status: 'pending' | 'generating' | 'completed' | 'failed';
  created: string;
}

export interface Transcript {
  id: string;
  sceneId: string;
  lines: DialogueLine[];
  generated: string;
  status: 'none' | 'generated' | 'customized';
}

export interface DialogueLine {
  id: string;
  characterId: string;
  text: string;
  emotion?: string;
  timing?: {
    start: number;
    duration: number;
  };
}

export type AnimationStyle = 
  | 'anime-classic' | 'shonen' | 'shojo' | 'seinen' 
  | 'cyberpunk' | 'fantasy' | 'mecha' | 'slice-of-life' 
  | 'dark-fantasy' | 'isekai';

export type CameraMovement = 
  | 'static' | 'pan-left' | 'pan-right' | 'pan-up' 
  | 'pan-down' | 'zoom-in' | 'zoom-out' | 'dolly-in' 
  | 'dolly-out' | 'orbit-left' | 'orbit-right';

export type Mood = 
  | 'happy' | 'sad' | 'tense' | 'peaceful' | 'exciting' 
  | 'mysterious' | 'romantic' | 'dramatic' | 'comedic' | 'dark';

export interface GenerationSettings {
  resolution: { width: number; height: number };
  fps: number;
  steps: number;
  cfgScale: number;
  sampler: string;
  scheduler: string;
  modelType: 'gguf' | 'checkpoint';
  modelPath: string;
  vaePath: string;
  clipPath: string;
  enableAudio: boolean;
  ttsEngine: 'web-speech' | 'coqui-tts' | 'elevenlabs';
  audioFormat: 'wav' | 'mp3' | 'aac';
}

export interface VoicePreset {
  id: string;
  name: string;
  description: string;
  voiceSettings: VoiceSettings;
  sample?: string;
}

STEP 7: Create Constants & Presets
-------------------------------------------
Create: src/constants/index.ts
Content:
import { VoicePreset } from '../types';

export const HARDWARE_PRESETS = {
  ultraFast: {
    name: '⚡ Ultra Fast',
    resolution: { width: 384, height: 256 },
    steps: 4,
    fps: 12,
    description: '30-60s per scene - Preview only'
  },
  fast: {
    name: '🚀 Fast',
    resolution: { width: 512, height: 384 },
    steps: 8,
    fps: 24,
    description: '1-2min per scene - Recommended for 6GB VRAM'
  },
  balanced: {
    name: '⚖️ Balanced',
    resolution: { width: 512, height: 384 },
    steps: 16,
    fps: 24,
    description: '2-4min per scene - Good quality'
  },
  quality: {
    name: '🎨 Quality',
    resolution: { width: 640, height: 480 },
    steps: 24,
    fps: 24,
    description: '3-6min per scene - High quality'
  },
  maximum: {
    name: '💎 Maximum',
    resolution: { width: 768, height: 576 },
    steps: 32,
    fps: 30,
    description: '6-12min per scene - May OOM on 6GB VRAM'
  }
};

export const VOICE_PRESETS: VoicePreset[] = [
  {
    id: 'young-male-hero',
    name: 'Young Male Hero',
    description: 'Energetic, youthful male voice',
    voiceSettings: {
      voiceId: 'Google US English Male',
      pitch: 1.1,
      rate: 1.0,
      volume: 1.0,
      style: 'energetic'
    }
  },
  {
    id: 'female-lead',
    name: 'Female Lead',
    description: 'Clear, expressive female voice',
    voiceSettings: {
      voiceId: 'Google US English Female',
      pitch: 1.2,
      rate: 0.95,
      volume: 1.0,
      style: 'expressive'
    }
  },
  {
    id: 'wise-mentor',
    name: 'Wise Mentor',
    description: 'Deep, calm older voice',
    voiceSettings: {
      voiceId: 'Microsoft David - English (United States)',
      pitch: 0.8,
      rate: 0.85,
      volume: 1.0,
      style: 'calm'
    }
  },
  {
    id: 'villain-dark',
    name: 'Villain (Dark)',
    description: 'Menacing, low-pitched voice',
    voiceSettings: {
      voiceId: 'Microsoft Mark - English (United States)',
      pitch: 0.7,
      rate: 0.9,
      volume: 1.0,
      style: 'menacing'
    }
  },
  {
    id: 'comedic-sidekick',
    name: 'Comedic Sidekick',
    description: 'Upbeat, fast-paced voice',
    voiceSettings: {
      voiceId: 'Google US English Male',
      pitch: 1.3,
      rate: 1.1,
      volume: 1.0,
      style: 'comedic'
    }
  },
  {
    id: 'robotic-ai',
    name: 'Robotic/AI',
    description: 'Synthetic, monotone voice',
    voiceSettings: {
      voiceId: 'Microsoft Zira - English (United States)',
      pitch: 0.9,
      rate: 1.0,
      volume: 1.0,
      style: 'monotone'
    }
  }
];

export const ANIMATION_STYLES: Record<AnimationStyle, { 
  name: string; 
  prompt: string;
  negative: string;
  description: string;
}> = {
  'anime-classic': {
    name: 'Anime Classic',
    prompt: 'anime style, cel shaded, vibrant colors, detailed line art, 2d animation',
    negative: '3d, cgi, photorealistic, blurry, low quality',
    description: 'Traditional cel-shaded anime'
  },
  'shonen': {
    name: 'Shonen',
    prompt: 'anime style, shonen manga, dynamic action, sharp details, intense colors, speed lines',
    negative: 'soft, romantic, blurry, still',
    description: 'Action-oriented, dynamic'
  },
  'shojo': {
    name: 'Shojo',
    prompt: 'anime style, shojo manga, soft colors, romantic lighting, delicate features, sparkles',
    negative: 'dark, gritty, action, harsh lighting',
    description: 'Romantic, soft colors'
  },
  'seinen': {
    name: 'Seinen',
    prompt: 'anime style, seinen manga, mature, detailed backgrounds, realistic proportions, moody lighting',
    negative: 'childish, simple, flat',
    description: 'Mature, detailed'
  },
  'cyberpunk': {
    name: 'Cyberpunk',
    prompt: 'anime style, cyberpunk, neon lights, dystopian, futuristic, cityscape, rain, holograms',
    negative: 'natural, rural, daytime, peaceful',
    description: 'Neon, futuristic'
  },
  'fantasy': {
    name: 'Fantasy',
    prompt: 'anime style, high fantasy, magic, mystical, enchanted, glowing effects, epic',
    negative: 'modern, sci-fi, mundane',
    description: 'Magical, mystical'
  },
  'mecha': {
    name: 'Mecha',
    prompt: 'anime style, mecha robots, mechanical details, giant robots, sci-fi, mechanical',
    negative: 'organic, natural, fantasy',
    description: 'Robots, sci-fi'
  },
  'slice-of-life': {
    name: 'Slice of Life',
    prompt: 'anime style, slice of life, cozy, everyday, warm lighting, domestic, school',
    negative: 'action, fantasy, dark, dramatic',
    description: 'Everyday, cozy'
  },
  'dark-fantasy': {
    name: 'Dark Fantasy',
    prompt: 'anime style, dark fantasy, gothic, shadows, dramatic, grim, supernatural horror',
    negative: 'bright, happy, comedic, modern',
    description: 'Gothic, shadowy'
  },
  'isekai': {
    name: 'Isekai',
    prompt: 'anime style, isekai, fantasy world, adventure, otherworldly, magical realm, rpg',
    negative: 'modern, realistic, mundane',
    description: 'Fantasy world adventure'
  }
};

export const CAMERA_MOVEMENTS: Record<CameraMovement, string> = {
  static: 'Static shot',
  'pan-left': 'Pan left (↔️)',
  'pan-right': 'Pan right (↔️)',
  'pan-up': 'Pan up (↕️)',
  'pan-down': 'Pan down (↕️)',
  'zoom-in': 'Zoom in (🔍+)',
  'zoom-out': 'Zoom out (🔍-)',
  'dolly-in': 'Dolly in (🎬+)',
  'dolly-out': 'Dolly out (🎬-)',
  'orbit-left': 'Orbit left (🔄)',
  'orbit-right': 'Orbit right (🔄)'
};

export const MOODS: Record<Mood, { 
  name: string; 
  prompt: string;
  audioStyle: string;
}> = {
  happy: { 
    name: '😊 Happy', 
    prompt: 'joyful, bright, cheerful, smiling',
    audioStyle: 'cheerful'
  },
  sad: { 
    name: '😢 Sad', 
    prompt: 'melancholic, tearful, somber, emotional',
    audioStyle: 'sad'
  },
  tense: { 
    name: '😰 Tense', 
    prompt: 'suspenseful, anxious, high tension, nervous',
    audioStyle: 'anxious'
  },
  peaceful: { 
    name: '😌 Peaceful', 
    prompt: 'calm, serene, tranquil, relaxing',
    audioStyle: 'calm'
  },
  exciting: { 
    name: '🤩 Exciting', 
    prompt: 'excited, energetic, enthusiastic, thrilled',
    audioStyle: 'excited'
  },
  mysterious: { 
    name: '🧐 Mysterious', 
    prompt: 'mysterious, enigmatic, puzzling, cryptic',
    audioStyle: 'mysterious'
  },
  romantic: { 
    name: '🥰 Romantic', 
    prompt: 'romantic, loving, affectionate, heartwarming',
    audioStyle: 'loving'
  },
  dramatic: { 
    name: '🎭 Dramatic', 
    prompt: 'dramatic, serious, intense, theatrical',
    audioStyle: 'dramatic'
  },
  comedic: { 
    name: '😄 Comedic', 
    prompt: 'funny, comedic, humorous, silly',
    audioStyle: 'funny'
  },
  dark: { 
    name: '👿 Dark', 
    prompt: 'dark, ominous, evil, sinister, malevolent',
    audioStyle: 'sinister'
  }
};

export const TTS_ENGINES = [
  { 
    id: 'web-speech', 
    name: 'Web Speech API', 
    description: 'Browser built-in, free, offline',
    requiresKey: false
  },
  { 
    id: 'coqui-tts', 
    name: 'Coqui TTS', 
    description: 'Local AI TTS server (recommended for quality)',
    requiresKey: false,
    localUrl: 'http://localhost:5002'
  },
  { 
    id: 'elevenlabs', 
    name: 'ElevenLabs', 
    description: 'Cloud API, ultra realistic',
    requiresKey: true,
    url: 'https://api.elevenlabs.io'
  }
];

export const DEFAULT_SETTINGS: GenerationSettings = {
  resolution: { width: 512, height: 384 },
  fps: 24,
  steps: 8,
  cfgScale: 1.0,
  sampler: 'euler',
  scheduler: 'normal',
  modelType: 'gguf',
  modelPath: 'models/unet/ltx-video-2b-Q4_K_M.gguf',
  vaePath: 'models/vae/ltx-video-2b-vae.safetensors',
  clipPath: 'models/clip/clip_l.safetensors',
  enableAudio: true,
  ttsEngine: 'web-speech',
  audioFormat: 'wav'
};

export const COMFYUI_NODES = {
  UNET_LOADER: 'UNETLoader',
  VAE_LOADER: 'VAELoader',
  DUAL_CLIP_LOADER: 'DualCLIPLoader',
  KSAMPLER: 'KSampler',
  LTX_VIDEO_DECODE: 'LTXVVideoDecoder',
  LTX_IMAGE_TO_VIDEO: 'LTXImageToVideo',
  SAVE_IMAGE: 'SaveImage',
  VHS_VIDEOCOMBINE: 'VHS_VideoCombine',
  VHS_LOADAUDIO: 'VHS_LoadAudio',
  VHS_AUDIOCONCAT: 'VHS_AudioConcat',
  VHS_COMBINE_AUDIO_VIDEO: 'VHS_COMBINE_AUDIO_VIDEO'
};

STEP 8: Create Storage Utilities
-------------------------------------------
Create: src/utils/storage.ts
Content:
import { Project, Character, Episode, Scene, GenerationSettings } from '../types';

const STORAGE_KEYS = {
  PROJECTS: 'ltx_anime_projects_v2',
  SETTINGS: 'ltx_anime_settings_v2',
  VOICES: 'ltx_anime_voice_samples_v2'
};

export const storage = {
  saveProjects: (projects: Project[]) => {
    try {
      localStorage.setItem(STORAGE_KEYS.PROJECTS, JSON.stringify(projects));
      return true;
    } catch (error) {
      console.error('Failed to save projects:', error);
      return false;
    }
  },

  loadProjects: (): Project[] => {
    try {
      const data = localStorage.getItem(STORAGE_KEYS.PROJECTS);
      return data ? JSON.parse(data) : [];
    } catch (error) {
      console.error('Failed to load projects:', error);
      return [];
    }
  },

  saveSettings: (settings: GenerationSettings) => {
    try {
      localStorage.setItem(STORAGE_KEYS.SETTINGS, JSON.stringify(settings));
      return true;
    } catch (error) {
      console.error('Failed to save settings:', error);
      return false;
    }
  },

  loadSettings: (): GenerationSettings | null => {
    try {
      const data = localStorage.getItem(STORAGE_KEYS.SETTINGS);
      return data ? JSON.parse(data) : null;
    } catch (error) {
      console.error('Failed to load settings:', error);
      return null;
    }
  },

  saveVoiceSample: (voiceId: string, audioData: string) => {
    try {
      const samples = storage.loadVoiceSamples();
      samples[voiceId] = audioData;
      localStorage.setItem(STORAGE_KEYS.VOICES, JSON.stringify(samples));
      return true;
    } catch (error) {
      console.error('Failed to save voice sample:', error);
      return false;
    }
  },

  loadVoiceSamples: (): Record<string, string> => {
    try {
      const data = localStorage.getItem(STORAGE_KEYS.VOICES);
      return data ? JSON.parse(data) : {};
    } catch (error) {
      console.error('Failed to load voice samples:', error);
      return {};
    }
  },

  exportProject: (project: Project, characters: Character[], episodes: Episode[]) => {
    return JSON.stringify({
      project,
      characters,
      episodes,
      version: '2.0.0',
      exported: new Date().toISOString()
    }, null, 2);
  },

  importProject: (jsonData: string) => {
    try {
      const data = JSON.parse(jsonData);
      if (!data.project || !data.characters || !data.episodes) {
        throw new Error('Invalid project format');
      }
      return data;
    } catch (error) {
      console.error('Failed to import project:', error);
      return null;
    }
  }
};

===============================================================================

PHASE 3: CORE HOOKS DEVELOPMENT
===============================================================================

STEP 9: Create Enhanced Logger Hook
-------------------------------------------
Create: src/hooks/useLogger.ts
Content:
import { useState, useCallback } from 'react';

export type LogLevel = 'info' | 'success' | 'warning' | 'error' | 'debug';
export type LogCategory = 'system' | 'comfyui' | 'ollama' | 'generation' | 'ai' | 'export' | 'audio';

export interface LogEntry {
  id: string;
  timestamp: Date;
  level: LogLevel;
  category: LogCategory;
  message: string;
  details?: string;
  solution?: string;
}

export const useLogger = () => {
  const [logs, setLogs] = useState<LogEntry[]>([]);
  const [maxLogs] = useState(1000);

  const addLog = useCallback((
    level: LogLevel,
    category: LogCategory,
    message: string,
    details?: string,
    solution?: string
  ) => {
    const entry: LogEntry = {
      id: Math.random().toString(36).substr(2, 9),
      timestamp: new Date(),
      level,
      category,
      message,
      details,
      solution
    };

    setLogs(prev => {
      const updated = [entry, ...prev];
      return updated.slice(0, maxLogs);
    });

    const consoleMethod = level === 'error' ? console.error : 
                         level === 'warning' ? console.warn : 
                         console.log;
    consoleMethod(`[${category.toUpperCase()}] ${message}`, details || '');
  }, [maxLogs]);

  const info = (category: LogCategory, message: string, details?: string) => 
    addLog('info', category, message, details);
  const success = (category: LogCategory, message: string, details?: string) => 
    addLog('success', category, message, details);
  const warning = (category: LogCategory, message: string, details?: string, solution?: string) => 
    addLog('warning', category, message, details, solution);
  const error = (category: LogCategory, message: string, details?: string, solution?: string) => 
    addLog('error', category, message, details, solution);
  const debug = (category: LogCategory, message: string, details?: string) => 
    addLog('debug', category, message, details);

  const clear = useCallback(() => setLogs([]), []);
  
  const exportLogs = useCallback(() => {
    const logText = logs
      .map(log => `[${log.timestamp.toISOString()}] [${log.level.toUpperCase()}] [${log.category.toUpperCase()}] ${log.message}${log.details ? '\n  Details: ' + log.details : ''}${log.solution ? '\n  Solution: ' + log.solution : ''}`)
      .join('\n');
    
    const blob = new Blob([logText], { type: 'text/plain' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `ltx-anime-logs-${new Date().toISOString().slice(0, 10)}.txt`;
    a.click();
    URL.revokeObjectURL(url);
  }, [logs]);

  const filterLogs = useCallback((level?: LogLevel, category?: LogCategory) => {
    return logs.filter(log => 
      (!level || log.level === level) && 
      (!category || log.category === category)
    );
  }, [logs]);

  return {
    logs,
    info,
    success,
    warning,
    error,
    debug,
    clear,
    exportLogs,
    filterLogs
  };
};

STEP 10: Create ComfyUI Integration Hook
-------------------------------------------
Create: src/hooks/useComfyUI.ts
Content:
import { useState, useCallback } from 'react';
import { useLogger } from './useLogger';
import { GenerationSettings } from '../types';
import { generateWorkflow } from '../utils/workflow';

export interface SystemInfo {
  gpuModel: string;
  vramTotal: number;
  vramFree: number;
  devices: any[];
  comfyuiVersion: string;
  pythonVersion: string;
  pytorchVersion: string;
  cudaAvailable: boolean;
  cudaVersion: string;
}

export interface QueueResponse {
  prompt_id: string;
  number: number;
  node_errors: Record<string, any>;
}

export interface HistoryResponse {
  status: {
    completed: boolean;
    failed: boolean;
  };
  outputs: Record<string, any>;
}

export const useComfyUI = (baseUrl: string = 'http://127.0.0.1:8188') => {
  const [connected, setConnected] = useState(false);
  const [checking, setChecking] = useState(false);
  const [systemInfo, setSystemInfo] = useState<SystemInfo | null>(null);
  const [queuePosition, setQueuePosition] = useState<number | null>(null);
  const { info, error, success, warning } = useLogger();

  const checkConnection = useCallback(async () => {
    setChecking(true);
    try {
      const response = await fetch(`${baseUrl}/system_stats`);
      if (!response.ok) throw new Error('Failed to connect');
      
      const data = await response.json();
      setSystemInfo({
        gpuModel: data.system?.gpu_model || 'Unknown',
        vramTotal: data.system?.vram_total || 0,
        vramFree: data.system?.vram_free || 0,
        devices: data.system?.devices || [],
        comfyuiVersion: data.system?.comfyui_version || 'Unknown',
        pythonVersion: data.system?.python_version || 'Unknown',
        pytorchVersion: data.system?.pytorch_version || 'Unknown',
        cudaAvailable: data.system?.cuda_enabled || false,
        cudaVersion: data.system?.cuda_version || 'Unknown'
      });
      
      setConnected(true);
      success('comfyui', 'Connected to ComfyUI', `GPU: ${data.system?.gpu_model}, VRAM: ${Math.round(data.system?.vram_total / 1024 / 1024)}MB`);
      
      await checkCustomNodes();
      
    } catch (err) {
      setConnected(false);
      error('comfyui', 'Failed to connect to ComfyUI', 
        `Could not reach ${baseUrl}. Make sure ComfyUI is running with --listen flag.`, 
        'Start ComfyUI: cd C:\\ComfyUI && venv\\Scripts\\activate && python main.py --lowvram --listen'
      );
    } finally {
      setChecking(false);
    }
  }, [baseUrl, success, error]);

  const checkCustomNodes = useCallback(async () => {
    try {
      const response = await fetch(`${baseUrl}/object_info`);
      const data = await response.json();
      
      const requiredNodes = ['LTXVVideoDecoder', 'LTXImageToVideo', 'VHS_VideoCombine'];
      const missingNodes = requiredNodes.filter(node => !data[node]);
      
      if (missingNodes.length > 0) {
        warning('comfyui', 'Missing required custom nodes', 
          `Missing: ${missingNodes.join(', ')}`, 
          `Install missing nodes via ComfyUI Manager or git clone`
        );
      } else {
        info('comfyui', 'All required nodes are installed');
      }
    } catch (err) {
      warning('comfyui', 'Could not verify custom nodes');
    }
  }, [baseUrl, warning, info]);

  const getNodeInfo = useCallback(async (nodeName: string) => {
    try {
      const response = await fetch(`${baseUrl}/object_info/${nodeName}`);
      return await response.json();
    } catch (err) {
      error('comfyui', `Failed to get node info for ${nodeName}`, err);
      return null;
    }
  }, [baseUrl, error]);

  const uploadImage = useCallback(async (file: File) => {
    const formData = new FormData();
    formData.append('image', file);
    
    try {
      const response = await fetch(`${baseUrl}/upload/image`, {
        method: 'POST',
        body: formData
      });
      
      if (!response.ok) throw new Error('Upload failed');
      return await response.json();
    } catch (err) {
      error('comfyui', 'Image upload failed', err);
      return null;
    }
  }, [baseUrl, error]);

  const queuePrompt = useCallback(async (
    scene: any, 
    settings: GenerationSettings,
    characters: any[],
    projectStyle: string,
    enableAudio: boolean = false,
    audioPath?: string
  ): Promise<QueueResponse | null> => {
    try {
      const workflow = generateWorkflow(scene, settings, characters, projectStyle, enableAudio, audioPath);
      
      const response = await fetch(`${baseUrl}/prompt`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ prompt: workflow })
      });
      
      if (!response.ok) {
        const errorData = await response.json().catch(() => ({}));
        throw new Error(errorData.message || 'Failed to queue prompt');
      }
      
      const data = await response.json();
      success('comfyui', 'Workflow queued successfully', `Prompt ID: ${data.prompt_id}`);
      return data;
      
    } catch (err) {
      error('comfyui', 'Failed to queue workflow', err);
      return null;
    }
  }, [baseUrl, success, error]);

  const checkProgress = useCallback(async (promptId: string) => {
    try {
      const response = await fetch(`${baseUrl}/history/${promptId}`);
      if (!response.ok) return null;
      
      const data = await response.json();
      return data[promptId] as HistoryResponse | undefined;
    } catch (err) {
      error('comfyui', 'Failed to check progress', err);
      return null;
    }
  }, [baseUrl, error]);

  const getQueueInfo = useCallback(async () => {
    try {
      const response = await fetch(`${baseUrl}/queue`);
      if (!response.ok) return null;
      
      const data = await response.json();
      setQueuePosition(data.queue_running.length + data.queue_pending.length);
      return data;
    } catch (err) {
      error('comfyui', 'Failed to get queue info', err);
      return null;
    }
  }, [baseUrl, error]);

  const interrupt = useCallback(async () => {
    try {
      await fetch(`${baseUrl}/interrupt`, { method: 'POST' });
      info('comfyui', 'Generation interrupted');
      return true;
    } catch (err) {
      error('comfyui', 'Failed to interrupt', err);
      return false;
    }
  }, [baseUrl, info, error]);

  const freeMemory = useCallback(async () => {
    try {
      await fetch(`${baseUrl}/free`, { method: 'POST' });
      info('comfyui', 'Memory freed');
      return true;
    } catch (err) {
      error('comfyui', 'Failed to free memory', err);
      return false;
    }
  }, [baseUrl, info, error]);

  return {
    connected,
    checking,
    systemInfo,
    queuePosition,
    checkConnection,
    getNodeInfo,
    uploadImage,
    queuePrompt,
    checkProgress,
    getQueueInfo,
    interrupt,
    freeMemory
  };
};

STEP 11: Create Ollama AI Integration Hook
-------------------------------------------
Create: src/hooks/useOllama.ts
Content:
import { useState, useCallback } from 'react';
import { useLogger } from './useLogger';
import { Character, Episode, Scene, Transcript, DialogueLine } from '../types';
import { VOICE_PRESETS } from '../constants';

export interface OllamaModel {
  name: string;
  model: string;
  size: number;
  modified_at: string;
  digest: string;
}

export interface OllamaGenerateRequest {
  model: string;
  prompt: string;
  system?: string;
  stream?: boolean;
  options?: {
    temperature?: number;
    top_p?: number;
    top_k?: number;
    repeat_penalty?: number;
    seed?: number;
  };
}

export const useOllama = (baseUrl: string = 'http://127.0.0.1:11434') => {
  const [connected, setConnected] = useState(false);
  const [checking, setChecking] = useState(false);
  const [models, setModels] = useState<OllamaModel[]>([]);
  const [selectedModel, setSelectedModel] = useState('llama3.2');
  const { info, error, success, warning } = useLogger();

  const checkConnection = useCallback(async () => {
    setChecking(true);
    try {
      const response = await fetch(`${baseUrl}/api/tags`);
      if (!response.ok) throw new Error('Failed to connect');
      
      const data = await response.json();
      setModels(data.models || []);
      
      if (data.models && data.models.length > 0) {
        setSelectedModel(data.models[0].name);
        setConnected(true);
        success('ollama', 'Connected to Ollama', `Available models: ${data.models.length}`);
      } else {
        warning('ollama', 'No models found', 'Pull a model first: ollama pull llama3.2');
      }
      
    } catch (err) {
      setConnected(false);
      error('ollama', 'Failed to connect to Ollama', 
        `Could not reach ${baseUrl}. Make sure Ollama is running.`, 
        'Start Ollama or run: ollama serve'
      );
    } finally {
      setChecking(false);
    }
  }, [baseUrl, success, error, warning]);

  const pullModel = useCallback(async (modelName: string) => {
    try {
      info('ollama', `Pulling model ${modelName}...`);
      const response = await fetch(`${baseUrl}/api/pull`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name: modelName })
      });
      
      if (!response.ok) throw new Error('Pull failed');
      success('ollama', `Model ${modelName} pulled successfully`);
      await checkConnection();
    } catch (err) {
      error('ollama', 'Failed to pull model', err);
    }
  }, [baseUrl, info, success, error, checkConnection]);

  const generate = useCallback(async (
    prompt: string, 
    system?: string,
    options: Partial<OllamaGenerateRequest['options']> = {}
  ): Promise<string | null> => {
    try {
      const response = await fetch(`${baseUrl}/api/generate`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          model: selectedModel,
          prompt,
          system,
          stream: false,
          options: {
            temperature: 0.7,
            top_p: 0.9,
            top_k: 40,
            repeat_penalty: 1.1,
            ...options
          }
        } as OllamaGenerateRequest)
      });
      
      if (!response.ok) {
        const errorData = await response.json().catch(() => ({}));
        throw new Error(errorData.error || 'Generation failed');
      }
      
      const data = await response.json();
      return data.response || null;
      
    } catch (err) {
      error('ollama', 'Generation failed', err);
      return null;
    }
  }, [baseUrl, selectedModel, error]);

  const generateCharacters = useCallback(async (
    project: any,
    count: number = 4
  ): Promise<Partial<Character>[]> => {
    const systemPrompt = `You are a creative anime character designer. Create unique, diverse characters with distinct personalities and appearances. Format each character as:

Name: [Character Name]
Role: [Main/Protagonist, Supporting, Antagonist, etc.]
Appearance: [Detailed physical description]
Personality: [Personality traits]
VoiceStyle: [Suggested voice type: young-male-hero, female-lead, wise-mentor, villain-dark, comedic-sidekick, robotic-ai]

Example:
Name: Akira Tanaka
Role: Protagonist
Appearance: Spiky black hair, determined brown eyes, athletic build, wears a red jacket
Personality: Brave, loyal, slightly impulsive, protective of friends
VoiceStyle: young-male-hero`;

    const userPrompt = `Create ${count} unique anime characters for a ${project.style} style series titled "${project.name}".
    ${project.description ? `Concept: ${project.description}` : ''}
    
    Make each character distinct with different roles, appearances, and personalities. Ensure they would work well together in a story.`;

    const response = await generate(userPrompt, systemPrompt, { temperature: 0.8 });
    
    if (!response) {
      error('ai', 'Failed to generate characters');
      return [];
    }

    const characters: Partial<Character>[] = [];
    const characterBlocks = response.split('\n\n').filter(block => block.trim());

    for (const block of characterBlocks) {
      const lines = block.split('\n').filter(line => line.trim());
      const char: Partial<Character> = {};

      for (const line of lines) {
        if (line.startsWith('Name:')) char.name = line.replace('Name:', '').trim();
        else if (line.startsWith('Role:')) char.role = line.replace('Role:', '').trim();
        else if (line.startsWith('Appearance:')) char.appearance = line.replace('Appearance:', '').trim();
        else if (line.startsWith('Personality:')) char.personality = line.replace('Personality:', '').trim();
        else if (line.startsWith('VoiceStyle:')) {
          const voiceStyle = line.replace('VoiceStyle:', '').trim();
          char.voiceSettings = mapVoiceStyleToPreset(voiceStyle);
        }
      }

      if (char.name && char.appearance) {
        char.consistencyPrompt = `${char.name}: ${char.appearance}`;
        characters.push(char);
      }
    }

    success('ai', `Generated ${characters.length} characters`);
    return characters;
  }, [generate, success, error]);

  const generateStory = useCallback(async (
    project: any,
    characters: any[],
    episodeNumber: number = 1
  ): Promise<Partial<Episode>> => {
    const charList = characters.map(c => `${c.name} (${c.role}): ${c.personality}`).join('\n');

    const systemPrompt = `You are an expert anime scriptwriter. Write engaging episode outlines with 6-8 scenes. Format each scene as:

Scene X: [Scene Title]
Description: [What happens visually]
Characters: [Who appears]
Mood: [happy/sad/tense/peaceful/exciting/mysterious/romantic/dramatic/comedic/dark]
Camera: [static/pan-left/pan-right/pan-up/pan-down/zoom-in/zoom-out/dolly-in/dolly-out/orbit-left/orbit-right]
Duration: [seconds]

Example:
Scene 1: Morning Encounter
Description: Akira rushes to school late, bumping into Sakura at the gate. They exchange awkward greetings.
Characters: Akira, Sakura
Mood: comedic
Camera: pan-right
Duration: 8`;

    const userPrompt = `Write Episode ${episodeNumber} for a ${project.style} anime series titled "${project.name}".
    ${project.description ? `Series concept: ${project.description}` : ''}
    
    Characters:
    ${charList}
    
    Create a compelling story with 6-8 scenes that develops the characters and advances the plot. Include dialogue opportunities.`;

    const response = await generate(userPrompt, systemPrompt, { temperature: 0.7 });
    
    if (!response) {
      error('ai', 'Failed to generate story');
      return { scenes: [] };
    }

    const scenes: Partial<Scene>[] = [];
    const sceneBlocks = response.split('\n\n').filter(block => block.trim());

    for (let i = 0; i < sceneBlocks.length; i++) {
      const block = sceneBlocks[i];
      const lines = block.split('\n').filter(line => line.trim());
      const scene: Partial<Scene> = { 
        title: `Scene ${i + 1}`, 
        characters: [] 
      };

      for (const line of lines) {
        if (line.startsWith(`Scene ${i + 1}:`) || line.startsWith('Scene:')) {
          scene.title = line.replace(/Scene \d+:|Scene:/, '').trim();
        }
        else if (line.startsWith('Description:')) scene.description = line.replace('Description:', '').trim();
        else if (line.startsWith('Characters:')) {
          const charNames = line.replace('Characters:', '').split(',').map(n => n.trim());
          scene.characters = charNames;
        }
        else if (line.startsWith('Mood:')) scene.mood = line.replace('Mood:', '').trim().toLowerCase();
        else if (line.startsWith('Camera:')) scene.camera = line.replace('Camera:', '').trim().toLowerCase();
        else if (line.startsWith('Duration:')) scene.duration = parseInt(line.replace('Duration:', '').trim());
      }

      if (scene.description) {
        scenes.push(scene);
      }
    }

    success('ai', `Generated episode with ${scenes.length} scenes`);
    return { 
      title: `Episode ${episodeNumber}`, 
      description: `AI-generated episode for ${project.name}`,
      scenes 
    };
  }, [generate, success, error]);

  const generateTranscript = useCallback(async (
    scene: any,
    characters: any[],
    projectStyle: string
  ): Promise<Transcript | null> => {
    const charList = characters.map(c => 
      `${c.name}: ${c.personality}, speaks like ${c.voiceSettings?.style || 'normal'}`
    ).join('\n');

    const systemPrompt = `You are an anime dialogue writer. Write natural, engaging dialogue for a ${projectStyle} scene. Format as:

Character: [Character Name]
Line: "[Dialogue text]"
Emotion: [emotion tag]
Timing: [start seconds] - [duration seconds]

Rules:
1. Keep dialogue concise (1-3 sentences per line)
2. Match each character's personality
3. Include natural pauses and reactions
4. Total duration should match scene duration
5. Use Japanese honorifics if appropriate (-san, -chan, -kun)
6. Include emotional subtext

Example:
Character: Akira
Line: "I'm not giving up! Not now, not ever!"
Emotion: determined
Timing: 0 - 3

Character: Sakura
Line: "Akira... your determination is amazing."
Emotion: admiring
Timing: 3.5 - 4`;

    const userPrompt = `Write dialogue for scene: "${scene.title}"
    Description: ${scene.description}
    Mood: ${scene.mood}
    Characters present: ${scene.characters?.join(', ') || 'Unknown'}
    Target duration: ${scene.duration || 10} seconds
    
    Characters:
    ${charList}
    
    Create engaging dialogue that fits the scene and characters.`;

    const response = await generate(userPrompt, systemPrompt, { 
      temperature: 0.8,
      top_p: 0.95
    });
    
    if (!response) {
      error('ai', 'Failed to generate transcript');
      return null;
    }

    const lines: any[] = [];
    const dialogueBlocks = response.split('\n\n').filter(block => block.trim());

    for (let i = 0; i < dialogueBlocks.length; i++) {
      const block = dialogueBlocks[i];
      const charMatch = block.match(/Character:\s*(.+)/);
      const lineMatch = block.match(/Line:\s*"(.+)"/);
      const emotionMatch = block.match(/Emotion:\s*(.+)/);
      const timingMatch = block.match(/Timing:\s*(\d+(?:\.\d+)?)\s*-\s*(\d+(?:\.\d+)?)/);

      if (charMatch && lineMatch) {
        const characterId = characters.find(c => 
          c.name.toLowerCase() === charMatch[1].trim().toLowerCase()
        )?.id;

        if (characterId) {
          lines.push({
            id: `line-${i}`,
            characterId,
            text: lineMatch[1],
            emotion: emotionMatch ? emotionMatch[1].trim() : undefined,
            timing: timingMatch ? {
              start: parseFloat(timingMatch[1]),
              duration: parseFloat(timingMatch[2])
            } : undefined
          });
        }
      }
    }

    success('ai', `Generated transcript with ${lines.length} lines`);
    return {
      id: `transcript-${Date.now()}`,
      sceneId: scene.id,
      lines,
      generated: new Date().toISOString(),
      status: lines.length > 0 ? 'generated' : 'none'
    };
  }, [generate, success, error]);

  const mapVoiceStyleToPreset = (style: string) => {
    const styleMap: Record<string, any> = {
      'young-male-hero': VOICE_PRESETS[0].voiceSettings,
      'female-lead': VOICE_PRESETS[1].voiceSettings,
      'wise-mentor': VOICE_PRESETS[2].voiceSettings,
      'villain-dark': VOICE_PRESETS[3].voiceSettings,
      'comedic-sidekick': VOICE_PRESETS[4].voiceSettings,
      'robotic-ai': VOICE_PRESETS[5].voiceSettings
    };
    return styleMap[style.toLowerCase().replace(/\s+/g, '-')] || VOICE_PRESETS[0].voiceSettings;
  };

  return {
    connected,
    checking,
    models,
    selectedModel,
    setSelectedModel,
    checkConnection,
    pullModel,
    generate,
    generateCharacters,
    generateStory,
    generateTranscript
  };
};

===============================================================================

PHASE 4: WORKFLOW & UTILITY FUNCTIONS
===============================================================================

STEP 12: Create ComfyUI Workflow Generator
-------------------------------------------
Create: src/utils/workflow.ts
Content:
import { GenerationSettings, Scene, Character } from '../types';
import { COMFYUI_NODES } from '../constants';

export const generateWorkflow = (
  scene: any,
  settings: GenerationSettings,
  characters: Character[],
  projectStyle: string,
  enableAudio: boolean = false,
  audioPath?: string
): any => {
  const workflow: any = {
    "1": {
      "inputs": {
        "unet_name": settings.modelPath.split('/').pop(),
        "weight_dtype": "default"
      },
      "class_type": settings.modelType === 'gguf' ? "UnetLoaderGGUF" : COMFYUI_NODES.UNET_LOADER,
      "_meta": { "title": "Load LTX-Video Model" }
    },
    "2": {
      "inputs": {
        "vae_name": settings.vaePath.split('/').pop()
      },
      "class_type": COMFYUI_NODES.VAE_LOADER,
      "_meta": { "title": "Load VAE" }
    },
    "3": {
      "inputs": {
        "clip_name1": settings.clipPath.split('/').pop(),
        "clip_name2": settings.clipPath.split('/').pop(),
        "type": "ltxv"
      },
      "class_type": COMFYUI_NODES.DUAL_CLIP_LOADER,
      "_meta": { "title": "Load CLIP" }
    },
    "4": {
      "inputs": {
        "text": buildPrompt(scene, characters, projectStyle),
        "clip": ["3", 0]
      },
      "class_type": "CLIPTextEncode",
      "_meta": { "title": "Positive Prompt" }
    },
    "5": {
      "inputs": {
        "text": ANIMATION_STYLES[projectStyle as any].negative,
        "clip": ["3", 0]
      },
      "class_type": "CLIPTextEncode",
      "_meta": { "title": "Negative Prompt" }
    },
    "6": {
      "inputs": {
        "width": settings.resolution.width,
        "height": settings.resolution.height,
        "length": Math.max(1, Math.floor((scene.duration || 8) * settings.fps / 8)),
        "batch_size": 1
      },
      "class_type": "EmptyLTXVLatentVideo",
      "_meta": { "title": "Empty Latent Video" }
    },
    "7": {
      "inputs": {
        "seed": Math.floor(Math.random() * 999999999999),
        "steps": settings.steps,
        "cfg": settings.cfgScale,
        "sampler_name": settings.sampler,
        "scheduler": settings.scheduler,
        "denoise": 1.0,
        "model": ["1", 0],
        "positive": ["4", 0],
        "negative": ["5", 0],
        "latent_image": ["6", 0]
      },
      "class_type": COMFYUI_NODES.KSAMPLER,
      "_meta": { "title": "KSampler" }
    },
    "8": {
      "inputs": {
        "latent": ["7", 0],
        "vae": ["2", 0]
      },
      "class_type": COMFYUI_NODES.LTX_VIDEO_DECODE,
      "_meta": { "title": "Decode LTX-Video Latent" }
    },
    "9": {
      "inputs": {
        "frame_rate": settings.fps,
        "loop_count": 0,
        "filename_prefix": `LTX-Video/${scene.id}`,
        "format": "video/h264-mp4",
        "pingpong": false,
        "save_output": true,
        "images": ["8", 0]
      },
      "class_type": COMFYUI_NODES.VHS_VIDEOCOMBINE,
      "_meta": { "title": "Video Combine" }
    }
  };

  if (enableAudio && audioPath) {
    workflow["10"] = {
      "inputs": {
        "audio": audioPath,
        "seek": 0
      },
      "class_type": COMFYUI_NODES.VHS_LOADAUDIO,
      "_meta": { "title": "Load Generated Audio" }
    };

    workflow["11"] = {
      "inputs": {
        "frame_rate": settings.fps,
        "loop_count": 0,
        "filename_prefix": `LTX-Video/${scene.id}-with-audio`,
        "format": "video/h264-mp4",
        "pingpong": false,
        "save_output": true,
        "audio": ["10", 0],
        "images": ["8", 0]
      },
      "class_type": COMFYUI_NODES.VHS_COMBINE_AUDIO_VIDEO,
      "_meta": { "title": "Combine Audio & Video" }
    };

    workflow["9"]["inputs"]["save_output"] = false;
  }

  if (!hasVHSNodes()) {
    delete workflow["9"];
    workflow["9"] = {
      "inputs": {
        "filename_prefix": `LTX-Video/${scene.id}`,
        "images": ["8", 0]
      },
      "class_type": COMFYUI_NODES.SAVE_IMAGE,
      "_meta": { "title": "Save Image Sequence" }
    };
  }

  return workflow;
};

const buildPrompt = (scene: any, characters: Character[], projectStyle: string): string => {
  const style = ANIMATION_STYLES[projectStyle as any];
  let prompt = style.prompt + ', ';
  
  if (scene.characters && scene.characters.length > 0) {
    const sceneChars = characters.filter(c => 
      scene.characters.includes(c.id) || scene.characters.includes(c.name)
    );
    
    for (const char of sceneChars) {
      prompt += `${char.consistencyPrompt}, `;
    }
  }
  
  prompt += `${scene.description}, ${MOODS[scene.mood as any].prompt}`;
  
  const cameraPrompts: Record<string, string> = {
    'pan-left': 'camera panning left',
    'pan-right': 'camera panning right',
    'pan-up': 'camera panning up',
    'pan-down': 'camera panning down',
    'zoom-in': 'camera zooming in',
    'zoom-out': 'camera zooming out',
    'dolly-in': 'camera dolly in',
    'dolly-out': 'camera dolly out',
    'orbit-left': 'camera orbiting left',
    'orbit-right': 'camera orbiting right'
  };
  
  if (scene.camera && cameraPrompts[scene.camera]) {
    prompt += `, ${cameraPrompts[scene.camera]}`;
  }
  
  return prompt.trim();
};

const hasVHSNodes = (): boolean => {
  return true;
};

export const generateAudioWorkflow = (
  transcript: any,
  characters: Character[],
  settings: GenerationSettings
): any => {
  const workflow: any = {};
  const audioNodes: string[] = [];

  transcript.lines.forEach((line: any, index: number) => {
    const character = characters.find(c => c.id === line.characterId);
    if (!character) return;

    const nodeId = (100 + index).toString();
    audioNodes.push(nodeId);

    if (settings.ttsEngine === 'web-speech') {
      workflow[nodeId] = {
        "inputs": {
          "text": line.text,
          "voice": character.voiceSettings.voiceId,
          "pitch": character.voiceSettings.pitch,
          "rate": character.voiceSettings.rate,
          "output": `audio/${transcript.sceneId}-${index}.wav`
        },
        "class_type": "WebSpeechTTS",
        "_meta": { "title": `TTS: ${character.name}` }
      };
    } else if (settings.ttsEngine === 'coqui-tts') {
      workflow[nodeId] = {
        "inputs": {
          "text": line.text,
          "speaker_id": character.voiceSettings.voiceId,
          "output": `audio/${transcript.sceneId}-${index}.wav`
        },
        "class_type": "CoquiTTS",
        "_meta": { "title": `TTS: ${character.name}` }
      };
    }
  });

  if (audioNodes.length > 0) {
    workflow["200"] = {
      "inputs": {
        "audio_files": audioNodes.map(node => [node, 0]),
        "output": `audio/${transcript.sceneId}-full.wav`
      },
      "class_type": "AudioConcat",
      "_meta": { "title": "Concatenate Audio" }
    };
  }

  return workflow;
};

===============================================================================

PHASE 5: UI COMPONENTS & MAIN APPLICATION
===============================================================================

STEP 13: Create Layout Components
-------------------------------------------
Create: src/components/layout/Header.tsx
Content:
import { HeaderProps } from '../../../types';

export const Header = ({ title, subtitle }: HeaderProps) => {
  return (
    <header className="bg-gray-900 border-b border-gray-800 py-6">
      <div className="container mx-auto px-4">
        <h1 className="text-3xl font-bold anime-gradient">{title}</h1>
        <p className="text-gray-400 mt-1">{subtitle}</p>
      </div>
    </header>
  );
};

Create: src/components/layout/Tabs.tsx
Content:
import { TabsProps } from '../../../types';

export const Tabs = ({ tabs, activeTab, onTabChange }: TabsProps) => {
  return (
    <div className="bg-gray-900 border-b border-gray-800 sticky top-0 z-10">
      <div className="container mx-auto px-4">
        <div className="flex gap-1 overflow-x-auto py-2 tab-grid">
          {tabs.map(tab => (
            <button
              key={tab.key}
              onClick={() => onTabChange(tab.key)}
              className={`flex items-center gap-2 px-4 py-2 rounded-lg transition-all whitespace-nowrap ${
                activeTab === tab.key
                  ? 'bg-blue-600 text-white'
                  : 'text-gray-400 hover:text-white hover:bg-gray-800'
              }`}
            >
              <span>{tab.icon}</span>
              <span className="font-medium">{tab.label}</span>
            </button>
          ))}
        </div>
      </div>
    </div>
  );
};

Create: src/components/layout/StatusBar.tsx
Content:
import { StatusBarProps } from '../../../types';

export const StatusBar = ({ comfyUIConnected, ollamaConnected, currentProject }: StatusBarProps) => {
  return (
    <footer className="bg-gray-900 border-t border-gray-800 mt-12 py-4">
      <div className="container mx-auto px-4">
        <div className="flex justify-between items-center text-sm">
          <div className="flex items-center gap-6">
            <div className="flex items-center gap-2">
              <div className={comfyUIConnected ? 'status-connected' : 'status-disconnected'} />
              <span>ComfyUI</span>
              <span className="text-gray-500">
                {comfyUIConnected ? 'Connected' : 'Disconnected'}
              </span>
            </div>
            
            <div className="flex items-center gap-2">
              <div className={ollamaConnected ? 'status-connected' : 'status-disconnected'} />
              <span>Ollama</span>
              <span className="text-gray-500">
                {ollamaConnected ? 'Connected' : 'Disconnected'}
              </span>
            </div>

            {currentProject && (
              <div className="flex items-center gap-2">
                <span>📁</span>
                <span className="truncate max-w-xs">{currentProject.name}</span>
              </div>
            )}
          </div>

          <div className="text-gray-500">
            LTX-2 Anime Agent v2.0.0 | Optimized for GTX 1660 Ti
          </div>
        </div>
      </div>
    </footer>
  );
};

STEP 14: Create Project Management Components
-------------------------------------------
Create: src/components/projects/Dashboard.tsx
Content:
import { useState } from 'react';
import { Project } from '../../types';
import { storage } from '../../utils/storage';

interface DashboardProps {
  projects: Project[];
  currentProject: Project | null;
  onSelectProject: (project: Project) => void;
  onCreateProject: (project: Partial<Project>) => void;
  onDeleteProject: (projectId: string) => void;
}

export const Dashboard = ({ projects, currentProject, onSelectProject, onCreateProject, onDeleteProject }: DashboardProps) => {
  const [showModal, setShowModal] = useState(false);
  const [newProject, setNewProject] = useState({ name: '', description: '', style: 'anime-classic' });

  const handleCreateProject = () => {
    if (newProject.name.trim()) {
      onCreateProject(newProject);
      setNewProject({ name: '', description: '', style: 'anime-classic' });
      setShowModal(false);
    }
  };

  const handleImportProject = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (event) => {
        try {
          const imported = storage.importProject(event.target?.result as string);
          if (imported) {
            onCreateProject(imported.project);
            // Would need to also import characters and episodes
            alert('Project imported successfully!');
          }
        } catch (err) {
          alert('Failed to import project');
        }
      };
      reader.readAsText(file);
    }
  };

  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <h1 className="text-3xl font-bold anime-gradient">Project Dashboard</h1>
        <div className="flex gap-2">
          <input
            type="file"
            accept=".json"
            onChange={handleImportProject}
            className="hidden"
            id="import-project"
          />
          <label
            htmlFor="import-project"
            className="px-4 py-2 bg-gray-700 hover:bg-gray-600 text-white rounded-lg cursor-pointer transition-colors"
          >
            📥 Import
          </label>
          <button
            onClick={() => setShowModal(true)}
            className="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-lg transition-colors"
          >
            ➕ New Project
          </button>
        </div>
      </div>

      {projects.length === 0 ? (
        <div className="bg-gray-900 p-12 rounded-lg border-2 border-dashed border-gray-700 text-center">
          <h2 className="text-xl font-semibold mb-2">No Projects Yet</h2>
          <p className="text-gray-400 mb-4">Create your first anime project to get started!</p>
          <button
            onClick={() => setShowModal(true)}
            className="px-6 py-3 bg-blue-600 hover:bg-blue-700 text-white rounded-lg transition-colors"
          >
            Create New Project
          </button>
        </div>
      ) : (
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          {projects.map(project => (
            <div
              key={project.id}
              onClick={() => onSelectProject(project)}
              className={`anime-card p-6 cursor-pointer ${
                currentProject?.id === project.id ? 'ring-2 ring-blue-500' : ''
              }`}
            >
              <h3 className="text-xl font-semibold mb-2">{project.name}</h3>
              <p className="text-gray-400 text-sm mb-4 line-clamp-2">
                {project.description || 'No description'}
              </p>
              <div className="flex justify-between items-center text-xs text-gray-500">
                <span>Style: {project.style}</span>
                <span>{new Date(project.created).toLocaleDateString()}</span>
              </div>
              <div className="flex gap-2 mt-4">
                <button
                  onClick={(e) => {
                    e.stopPropagation();
                    onSelectProject(project);
                  }}
                  className="flex-1 px-3 py-1 bg-blue-600 hover:bg-blue-700 text-white rounded text-sm transition-colors"
                >
                  Open
                </button>
                <button
                  onClick={(e) => {
                    e.stopPropagation();
                    if (confirm('Delete this project?')) {
                      onDeleteProject(project.id);
                    }
                  }}
                  className="px-3 py-1 bg-red-600 hover:bg-red-700 text-white rounded text-sm transition-colors"
                >
                  Delete
                </button>
              </div>
            </div>
          ))}
        </div>
      )}

      {showModal && (
        <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
          <div className="bg-gray-800 p-6 rounded-lg max-w-md w-full">
            <h2 className="text-2xl font-bold mb-4">Create New Project</h2>
            <div className="space-y-4">
              <div>
                <label className="block text-sm font-medium mb-2">Project Name</label>
                <input
                  type="text"
                  value={newProject.name}
                  onChange={(e) => setNewProject(prev => ({ ...prev, name: e.target.value }))}
                  className="w-full px-3 py-2 border rounded-lg bg-gray-700 border-gray-600 text-white"
                  placeholder="My Anime Series"
                />
              </div>
              <div>
                <label className="block text-sm font-medium mb-2">Description</label>
                <textarea
                  value={newProject.description}
                  onChange={(e) => setNewProject(prev => ({ ...prev, description: e.target.value }))}
                  className="w-full px-3 py-2 border rounded-lg bg-gray-700 border-gray-600 text-white"
                  rows={3}
                  placeholder="Story concept..."
                />
              </div>
              <div>
                <label className="block text-sm font-medium mb-2">Animation Style</label>
                <select
                  value={newProject.style}
                  onChange={(e) => setNewProject(prev => ({ ...prev, style: e.target.value }))}
                  className="w-full px-3 py-2 border rounded-lg bg-gray-700 border-gray-600 text-white"
                >
                  {Object.entries(ANIMATION_STYLES).map(([key, style]) => (
                    <option key={key} value={key}>{style.name}</option>
                  ))}
                </select>
              </div>
              <div className="flex gap-3">
                <button
                  onClick={handleCreateProject}
                  className="flex-1 bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded-lg transition-colors"
                >
                  Create
                </button>
                <button
                  onClick={() => setShowModal(false)}
                  className="flex-1 bg-gray-700 hover:bg-gray-600 text-white py-2 px-4 rounded-lg transition-colors"
                >
                  Cancel
                </button>
              </div>
            </div>
          </div>
        </div>
      )}
    </div>
  );
};

STEP 15: Create Character Management Components
-------------------------------------------
Create: src/components/characters/CharacterManager.tsx
Content:
import { useState } from 'react';
import { Character, Project } from '../../types';
import { CharacterEditor } from './CharacterEditor';
import { useOllama } from '../../hooks/useOllama';
import { useLogger } from '../../hooks/useLogger';

interface CharacterManagerProps {
  project: Project;
  characters: Character[];
  onCreateCharacter: (character: Partial<Character>) => void;
  onUpdateCharacter: (characterId: string, updates: Partial<Character>) => void;
  onDeleteCharacter: (characterId: string) => void;
  onGenerateCharacters: () => void;
}

export const CharacterManager = ({ 
  project, 
  characters, 
  onCreateCharacter,
  onUpdateCharacter,
  onDeleteCharacter,
  onGenerateCharacters
}: CharacterManagerProps) => {
  const [editingCharacter, setEditingCharacter] = useState<Character | null>(null);
  const [showEditor, setShowEditor] = useState(false);

  const handleAIGenerate = async () => {
    await onGenerateCharacters();
  };

  const handleEdit = (character: Character) => {
    setEditingCharacter(character);
    setShowEditor(true);
  };

  const handleSave = (character: Character) => {
    if (editingCharacter) {
      onUpdateCharacter(character.id, character);
    } else {
      onCreateCharacter(character);
    }
    setShowEditor(false);
    setEditingCharacter(null);
  };

  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <h1 className="text-3xl font-bold anime-gradient">Character Manager</h1>
        <div className="flex gap-2">
          <button
            onClick={handleAIGenerate}
            className="px-4 py-2 bg-purple-600 hover:bg-purple-700 text-white rounded-lg transition-colors"
          >
            🤖 AI Generate
          </button>
          <button
            onClick={() => {
              setEditingCharacter(null);
              setShowEditor(true);
            }}
            className="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-lg transition-colors"
          >
            ➕ Add Character
          </button>
        </div>
      </div>

      {characters.length === 0 ? (
        <div className="bg-gray-900 p-12 rounded-lg border-2 border-dashed border-gray-700 text-center">
          <h2 className="text-xl font-semibold mb-2">No Characters Yet</h2>
          <p className="text-gray-400 mb-4">Add characters manually or use AI generation!</p>
          <button
            onClick={handleAIGenerate}
            className="px-6 py-3 bg-purple-600 hover:bg-purple-700 text-white rounded-lg transition-colors"
          >
            🤖 Generate Characters with AI
          </button>
        </div>
      ) : (
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          {characters.map(character => (
            <div key={character.id} className="anime-card p-6">
              <div className="flex justify-between items-start mb-4">
                <div>
                  <h3 className="text-xl font-semibold">{character.name}</h3>
                  <p className="text-sm text-gray-400">{character.role}</p>
                </div>
                <span className={`px-2 py-1 rounded text-xs ${
                  character.role === 'Protagonist' ? 'bg-blue-700' :
                  character.role === 'Antagonist' ? 'bg-red-700' :
                  'bg-gray-700'
                }`}>
                  {character.role}
                </span>
              </div>
              
              <div className="space-y-2 text-sm">
                <div>
                  <span className="text-gray-400">Appearance:</span>
                  <p className="text-gray-300 line-clamp-2">{character.appearance}</p>
                </div>
                <div>
                  <span className="text-gray-400">Personality:</span>
                  <p className="text-gray-300 line-clamp-2">{character.personality}</p>
                </div>
                <div>
                  <span className="text-gray-400">Voice:</span>
                  <p className="text-gray-300">{character.voiceSettings?.voiceId || 'Default'}</p>
                </div>
              </div>

              <div className="flex gap-2 mt-4">
                <button
                  onClick={() => handleEdit(character)}
                  className="flex-1 px-3 py-1 bg-blue-600 hover:bg-blue-700 text-white rounded text-sm transition-colors"
                >
                  Edit
                </button>
                <button
                  onClick={() => {
                    if (confirm(`Delete ${character.name}?`)) {
                      onDeleteCharacter(character.id);
                    }
                  }}
                  className="px-3 py-1 bg-red-600 hover:bg-red-700 text-white rounded text-sm transition-colors"
                >
                  Delete
                </button>
              </div>
            </div>
          ))}
        </div>
      )}

      {showEditor && (
        <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
          <div className="bg-gray-800 p-6 rounded-lg max-w-2xl w-full max-h-screen overflow-y-auto">
            <CharacterEditor
              character={editingCharacter}
              onSave={handleSave}
              onCancel={() => {
                setShowEditor(false);
                setEditingCharacter(null);
              }}
            />
          </div>
        </div>
      )}
    </div>
  );
};

Create: src/components/characters/CharacterEditor.tsx
Content:
import { useState } from 'react';
import { Character, VoicePreset, VoiceSettings } from '../../types';
import { VOICE_PRESETS } from '../../constants';
import { VoiceSettingsPanel } from './VoiceSettingsPanel';

interface CharacterEditorProps {
  character?: Character;
  onSave: (character: Character) => void;
  onCancel: () => void;
}

export const CharacterEditor = ({ character, onSave, onCancel }: CharacterEditorProps) => {
  const [formData, setFormData] = useState({
    name: character?.name || '',
    appearance: character?.appearance || '',
    personality: character?.personality || '',
    role: character?.role || 'Supporting',
    consistencyPrompt: character?.consistencyPrompt || '',
    voiceSettings: character?.voiceSettings || VOICE_PRESETS[0].voiceSettings
  });

  const [selectedPreset, setSelectedPreset] = useState<string>('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSave({
      ...character,
      ...formData,
      id: character?.id || Date.now().toString(),
      created: character?.created || new Date().toISOString()
    } as Character);
  };

  const applyVoicePreset = (presetId: string) => {
    const preset = VOICE_PRESETS.find(p => p.id === presetId);
    if (preset) {
      setFormData(prev => ({ ...prev, voiceSettings: preset.voiceSettings }));
      setSelectedPreset(presetId);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-6">
      <div>
        <label className="block text-sm font-medium mb-2">Name</label>
        <input
          type="text"
          value={formData.name}
          onChange={e => setFormData(prev => ({ ...prev, name: e.target.value }))}
          className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
          required
        />
      </div>

      <div>
        <label className="block text-sm font-medium mb-2">Role</label>
        <select
          value={formData.role}
          onChange={e => setFormData(prev => ({ ...prev, role: e.target.value }))}
          className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
        >
          <option value="Protagonist">Protagonist</option>
          <option value="Main">Main Character</option>
          <option value="Supporting">Supporting</option>
          <option value="Antagonist">Antagonist</option>
          <option value="Guest">Guest</option>
        </select>
      </div>

      <div>
        <label className="block text-sm font-medium mb-2">Appearance</label>
        <textarea
          value={formData.appearance}
          onChange={e => setFormData(prev => ({ ...prev, appearance: e.target.value }))}
          className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
          rows={3}
          placeholder="Hair, eyes, build, clothing..."
          required
        />
      </div>

      <div>
        <label className="block text-sm font-medium mb-2">Personality</label>
        <textarea
          value={formData.personality}
          onChange={e => setFormData(prev => ({ ...prev, personality: e.target.value }))}
          className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
          rows={2}
          placeholder="Traits, quirks, behavior..."
        />
      </div>

      <div className="border-t border-gray-700 pt-6">
        <h3 className="text-lg font-semibold mb-4 flex items-center gap-2">
          🎤 Voice Settings
        </h3>
        
        <div className="mb-4">
          <label className="block text-sm font-medium mb-2">Voice Preset</label>
          <select
            value={selectedPreset}
            onChange={e => applyVoicePreset(e.target.value)}
            className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
          >
            <option value="">Select a preset...</option>
            {VOICE_PRESETS.map(preset => (
              <option key={preset.id} value={preset.id}>
                {preset.name} - {preset.description}
              </option>
            ))}
          </select>
        </div>

        <VoiceSettingsPanel
          voiceSettings={formData.voiceSettings}
          onChange={newSettings => setFormData(prev => ({ ...prev, voiceSettings: newSettings }))}
        />
      </div>

      <div>
        <label className="block text-sm font-medium mb-2">Consistency Prompt</label>
        <textarea
          value={formData.consistencyPrompt}
          onChange={e => setFormData(prev => ({ ...prev, consistencyPrompt: e.target.value }))}
          className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
          rows={2}
          placeholder="Auto-generated from name + appearance"
        />
        <p className="text-xs text-gray-400 mt-1">
          This helps maintain character consistency across scenes
        </p>
      </div>

      <div className="flex gap-3">
        <button
          type="submit"
          className="flex-1 bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded-lg transition-colors"
        >
          {character ? 'Update Character' : 'Create Character'}
        </button>
        <button
          type="button"
          onClick={onCancel}
          className="flex-1 bg-gray-700 hover:bg-gray-600 text-white py-2 px-4 rounded-lg transition-colors"
        >
          Cancel
        </button>
      </div>
    </form>
  );
};

Create: src/components/characters/VoiceSettingsPanel.tsx
Content:
import { VoiceSettings } from '../../types';

interface VoiceSettingsPanelProps {
  voiceSettings: VoiceSettings;
  onChange: (settings: VoiceSettings) => void;
}

export const VoiceSettingsPanel = ({ voiceSettings, onChange }: VoiceSettingsPanelProps) => {
  const handleChange = (key: keyof VoiceSettings, value: any) => {
    onChange({ ...voiceSettings, [key]: value });
  };

  return (
    <div className="space-y-4 bg-gray-900 p-4 rounded-lg">
      <div>
        <label className="block text-sm font-medium mb-1">Voice ID</label>
        <input
          type="text"
          value={voiceSettings.voiceId}
          onChange={e => handleChange('voiceId', e.target.value)}
          className="w-full px-3 py-1 border rounded bg-gray-800 border-gray-700 text-white text-sm"
          placeholder="Google US English Male"
        />
      </div>

      <div className="grid grid-cols-3 gap-3">
        <div>
          <label className="block text-sm font-medium mb-1">
            Pitch: {voiceSettings.pitch.toFixed(1)}
          </label>
          <input
            type="range"
            min="0.5"
            max="2.0"
            step="0.1"
            value={voiceSettings.pitch}
            onChange={e => handleChange('pitch', parseFloat(e.target.value))}
            className="w-full"
          />
        </div>

        <div>
          <label className="block text-sm font-medium mb-1">
            Rate: {voiceSettings.rate.toFixed(1)}
          </label>
          <input
            type="range"
            min="0.5"
            max="2.0"
            step="0.1"
            value={voiceSettings.rate}
            onChange={e => handleChange('rate', parseFloat(e.target.value))}
            className="w-full"
          />
        </div>

        <div>
          <label className="block text-sm font-medium mb-1">
            Volume: {(voiceSettings.volume * 100).toFixed(0)}%
          </label>
          <input
            type="range"
            min="0"
            max="1"
            step="0.05"
            value={voiceSettings.volume}
            onChange={e => handleChange('volume', parseFloat(e.target.value))}
            className="w-full"
          />
        </div>
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Style/Emotion Hint</label>
        <input
          type="text"
          value={voiceSettings.style || ''}
          onChange={e => handleChange('style', e.target.value)}
          className="w-full px-3 py-1 border rounded bg-gray-800 border-gray-700 text-white text-sm"
          placeholder="energetic, calm, menacing, etc."
        />
      </div>

      <div className="flex gap-2 mt-4">
        <button
          type="button"
          onClick={() => {
            const utterance = new SpeechSynthesisUtterance("Hello, this is my voice!");
            utterance.voice = speechSynthesis.getVoices().find(v => v.name === voiceSettings.voiceId) || null;
            utterance.pitch = voiceSettings.pitch;
            utterance.rate = voiceSettings.rate;
            utterance.volume = voiceSettings.volume;
            speechSynthesis.speak(utterance);
          }}
          className="px-3 py-1 bg-purple-600 hover:bg-purple-700 text-white rounded text-sm transition-colors"
        >
          🔊 Test Voice
        </button>
        <button
          type="button"
          onClick={() => {
            speechSynthesis.getVoices();
          }}
          className="px-3 py-1 bg-gray-700 hover:bg-gray-600 text-white rounded text-sm transition-colors"
        >
          🔄 Load Voices
        </button>
      </div>
    </div>
  );
};

STEP 16: Create Episode & Scene Management Components
-------------------------------------------
Create: src/components/episodes/EpisodeManager.tsx
Content:
import { useState } from 'react';
import { Episode, Scene, Character, Project } from '../../types';
import { SceneEditor } from './SceneEditor';
import { useOllama } from '../../hooks/useOllama';
import { useLogger } from '../../hooks/useLogger';

interface EpisodeManagerProps {
  project: Project;
  episodes: Episode[];
  characters: Character[];
  onCreateEpisode: (episode: Partial<Episode>) => void;
  onUpdateEpisode: (episodeId: string, updates: Partial<Episode>) => void;
  onGenerateStory: () => void;
  onGenerateTranscript: (scene: Scene) => void;
}

export const EpisodeManager = ({ 
  project, 
  episodes, 
  characters,
  onCreateEpisode,
  onUpdateEpisode,
  onGenerateStory,
  onGenerateTranscript
}: EpisodeManagerProps) => {
  const [editingScene, setEditingScene] = useState<{ episode: Episode, scene: Scene | null } | null>(null);
  const ollama = useOllama();
  const { info } = useLogger();

  const handleCreateScene = (episode: Episode) => {
    setEditingScene({ episode, scene: null });
  };

  const handleEditScene = (episode: Episode, scene: Scene) => {
    setEditingScene({ episode, scene });
  };

  const handleSaveScene = (sceneData: Scene) => {
    if (editingScene) {
      const { episode, scene } = editingScene;
      const updatedScenes = scene 
        ? episode.scenes.map(s => s.id === scene.id ? sceneData : s)
        : [...episode.scenes, sceneData];
      
      onUpdateEpisode(episode.id, { scenes: updatedScenes });
      setEditingScene(null);
    }
  };

  const handleAIGenerate = async () => {
    await onGenerateStory();
  };

  if (episodes.length === 0) {
    return (
      <div className="space-y-6">
        <div className="flex justify-between items-center">
          <h1 className="text-3xl font-bold anime-gradient">Episode Manager</h1>
          <button
            onClick={handleAIGenerate}
            className="px-4 py-2 bg-purple-600 hover:bg-purple-700 text-white rounded-lg transition-colors"
          >
            🤖 AI Generate Episode
          </button>
        </div>
        <div className="bg-gray-900 p-12 rounded-lg border-2 border-dashed border-gray-700 text-center">
          <h2 className="text-xl font-semibold mb-2">No Episodes Yet</h2>
          <p className="text-gray-400 mb-4">Generate your first episode with AI!</p>
          <button
            onClick={handleAIGenerate}
            className="px-6 py-3 bg-purple-600 hover:bg-purple-700 text-white rounded-lg transition-colors"
          >
            🤖 Generate Episode with AI
          </button>
        </div>
      </div>
    );
  }

  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <h1 className="text-3xl font-bold anime-gradient">Episode Manager</h1>
        <button
          onClick={handleAIGenerate}
          className="px-4 py-2 bg-purple-600 hover:bg-purple-700 text-white rounded-lg transition-colors"
        >
          🤖 AI Generate Episode
        </button>
      </div>

      {episodes.map(episode => (
        <div key={episode.id} className="bg-gray-900 rounded-lg p-6 border border-gray-700">
          <div className="flex justify-between items-center mb-4">
            <div>
              <h2 className="text-2xl font-bold">{episode.title}</h2>
              <p className="text-gray-400 mt-1">{episode.description}</p>
            </div>
            <div className="text-sm text-gray-500">
              {episode.scenes.length} scenes
            </div>
          </div>

          <div className="space-y-4">
            {episode.scenes.length === 0 ? (
              <div className="bg-gray-800 p-8 rounded-lg border-2 border-dashed border-gray-700 text-center">
                <p className="text-gray-400 mb-4">No scenes in this episode yet</p>
                <button
                  onClick={() => handleCreateScene(episode)}
                  className="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-lg transition-colors"
                >
                  + Add Scene
                </button>
              </div>
            ) : (
              <div className="space-y-3">
                {episode.scenes.map((scene, index) => (
                  <div key={scene.id} className="bg-gray-800 p-4 rounded-lg border border-gray-700">
                    <div className="flex justify-between items-start">
                      <div className="flex-1">
                        <div className="flex items-center gap-2">
                          <h3 className="text-lg font-semibold">
                            Scene {index + 1}: {scene.title}
                          </h3>
                          <span className={`px-2 py-1 rounded text-xs ${
                            scene.status === 'completed' ? 'bg-green-700' :
                            scene.status === 'generating' ? 'bg-yellow-700' :
                            'bg-gray-700'
                          }`}>
                            {scene.status}
                          </span>
                          {scene.transcript && (
                            <span className="px-2 py-1 bg-purple-700 bg-opacity-50 rounded text-xs">
                              📝
                            </span>
                          )}
                        </div>
                        <p className="text-gray-400 text-sm mt-1">{scene.description}</p>
                        <div className="flex gap-4 mt-2 text-xs text-gray-500">
                          <span>Mood: {scene.mood}</span>
                          <span>Camera: {scene.camera}</span>
                          <span>Duration: {scene.duration}s</span>
                          <span>Characters: {scene.characters.length}</span>
                        </div>
                      </div>
                      <div className="flex gap-2 ml-4">
                        <button
                          onClick={() => onGenerateTranscript(scene)}
                          className="px-3 py-1 bg-purple-600 hover:bg-purple-700 text-white rounded text-sm transition-colors"
                          title="Generate Transcript"
                        >
                          📝
                        </button>
                        <button
                          onClick={() => handleEditScene(episode, scene)}
                          className="px-3 py-1 bg-blue-600 hover:bg-blue-700 text-white rounded text-sm transition-colors"
                        >
                          Edit
                        </button>
                        <button
                          onClick={() => {
                            const updatedScenes = episode.scenes.filter(s => s.id !== scene.id);
                            onUpdateEpisode(episode.id, { scenes: updatedScenes });
                          }}
                          className="px-3 py-1 bg-red-600 hover:bg-red-700 text-white rounded text-sm transition-colors"
                        >
                          Delete
                        </button>
                      </div>
                    </div>
                  </div>
                ))}
              </div>
            )}

            <button
              onClick={() => handleCreateScene(episode)}
              className="w-full py-3 border-2 border-dashed border-gray-600 hover:border-gray-500 rounded-lg text-gray-400 hover:text-white transition-colors"
            >
              + Add New Scene
            </button>
          </div>
        </div>
      ))}

      {editingScene && (
        <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
          <div className="bg-gray-800 p-6 rounded-lg max-w-4xl w-full max-h-screen overflow-y-auto">
            <SceneEditor
              scene={editingScene.scene}
              characters={characters}
              onSave={handleSaveScene}
              onCancel={() => setEditingScene(null)}
              onGenerateTranscript={onGenerateTranscript}
              transcript={editingScene.scene?.transcript}
            />
          </div>
        </div>
      )}
    </div>
  );
};

Create: src/components/episodes/SceneEditor.tsx
Content:
import { useState } from 'react';
import { Scene, Character, Mood, CameraMovement, Transcript } from '../../types';
import { CAMERA_MOVEMENTS, MOODS } from '../../constants';
import { TranscriptEditor } from '../transcripts/TranscriptEditor';

interface SceneEditorProps {
  scene?: Scene;
  characters: Character[];
  onSave: (scene: Scene) => void;
  onCancel: () => void;
  onGenerateTranscript?: (scene: Scene) => void;
  transcript?: Transcript;
}

export const SceneEditor = ({ 
  scene, 
  characters, 
  onSave, 
  onCancel,
  onGenerateTranscript,
  transcript
}: SceneEditorProps) => {
  const [formData, setFormData] = useState({
    title: scene?.title || '',
    description: scene?.description || '',
    mood: scene?.mood || 'neutral',
    camera: scene?.camera || 'static',
    duration: scene?.duration || 8,
    characters: scene?.characters || [],
    style: scene?.style || ''
  });

  const [showTranscript, setShowTranscript] = useState(false);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSave({
      ...scene,
      ...formData,
      id: scene?.id || Date.now().toString(),
      created: scene?.created || new Date().toISOString(),
      status: scene?.status || 'pending'
    } as Scene);
  };

  const toggleCharacter = (characterId: string) => {
    setFormData(prev => ({
      ...prev,
      characters: prev.characters.includes(characterId)
        ? prev.characters.filter(id => id !== characterId)
        : [...prev.characters, characterId]
    }));
  };

  return (
    <div className="space-y-6">
      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label className="block text-sm font-medium mb-2">Scene Title</label>
          <input
            type="text"
            value={formData.title}
            onChange={e => setFormData(prev => ({ ...prev, title: e.target.value }))}
            className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
            required
          />
        </div>

        <div>
          <label className="block text-sm font-medium mb-2">Description</label>
          <textarea
            value={formData.description}
            onChange={e => setFormData(prev => ({ ...prev, description: e.target.value }))}
            className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
            rows={3}
            placeholder="What happens in this scene..."
            required
          />
        </div>

        <div className="grid grid-cols-2 gap-4">
          <div>
            <label className="block text-sm font-medium mb-2">Mood</label>
            <select
              value={formData.mood}
              onChange={e => setFormData(prev => ({ ...prev, mood: e.target.value }))}
              className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
            >
              {Object.entries(MOODS).map(([key, mood]) => (
                <option key={key} value={key}>{mood.name}</option>
              ))}
            </select>
          </div>

          <div>
            <label className="block text-sm font-medium mb-2">Camera</label>
            <select
              value={formData.camera}
              onChange={e => setFormData(prev => ({ ...prev, camera: e.target.value }))}
              className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
            >
              {Object.entries(CAMERA_MOVEMENTS).map(([key, name]) => (
                <option key={key} value={key}>{name}</option>
              ))}
            </select>
          </div>
        </div>

        <div>
          <label className="block text-sm font-medium mb-2">
            Duration: {formData.duration} seconds
          </label>
          <input
            type="range"
            min="4"
            max="30"
            value={formData.duration}
            onChange={e => setFormData(prev => ({ ...prev, duration: parseInt(e.target.value) }))}
            className="w-full"
          />
          <p className="text-xs text-gray-400 mt-1">
            Longer durations require more VRAM and time
          </p>
        </div>

        <div>
          <label className="block text-sm font-medium mb-2">Characters in Scene</label>
          <div className="space-y-2 max-h-32 overflow-y-auto">
            {characters.map(character => (
              <label key={character.id} className="flex items-center gap-2">
                <input
                  type="checkbox"
                  checked={formData.characters.includes(character.id)}
                  onChange={() => toggleCharacter(character.id)}
                  className="rounded"
                />
                <span>{character.name}</span>
                <span className="text-xs text-gray-400">({character.role})</span>
              </label>
            ))}
          </div>
        </div>

        <div className="flex gap-3 pt-4">
          <button
            type="submit"
            className="flex-1 bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded-lg transition-colors"
          >
            {scene ? 'Update Scene' : 'Create Scene'}
          </button>
          <button
            type="button"
            onClick={onCancel}
            className="flex-1 bg-gray-700 hover:bg-gray-600 text-white py-2 px-4 rounded-lg transition-colors"
          >
            Cancel
          </button>
        </div>
      </form>

      {scene && (
        <div className="border-t border-gray-700 pt-6">
          <div className="flex justify-between items-center mb-4">
            <h3 className="text-lg font-semibold">📝 Transcript</h3>
            <div className="flex gap-2">
              {onGenerateTranscript && (
                <button
                  onClick={() => onGenerateTranscript(scene)}
                  className="px-3 py-1 bg-purple-600 hover:bg-purple-700 text-white rounded text-sm transition-colors"
                >
                  🤖 AI Generate
                </button>
              )}
              <button
                onClick={() => setShowTranscript(!showTranscript)}
                className="px-3 py-1 bg-gray-700 hover:bg-gray-600 text-white rounded text-sm transition-colors"
              >
                {showTranscript ? 'Hide' : 'Show'} Transcript
              </button>
            </div>
          </div>

          {showTranscript && transcript && (
            <TranscriptEditor
              transcript={transcript}
              characters={characters}
              onChange={() => {}}
            />
          )}
        </div>
      )}
    </div>
  );
};

Create: src/components/transcripts/TranscriptEditor.tsx
Content:
import { useState, useEffect } from 'react';
import { Transcript, DialogueLine, Character } from '../../types';

interface TranscriptEditorProps {
  transcript: Transcript;
  characters: Character[];
  onChange: (transcript: Transcript) => void;
}

export const TranscriptEditor = ({ transcript, characters, onChange }: TranscriptEditorProps) => {
  const [lines, setLines] = useState<DialogueLine[]>(transcript.lines);
  const [editingId, setEditingId] = useState<string | null>(null);
  const [editedText, setEditedText] = useState('');

  useEffect(() => {
    setLines(transcript.lines);
  }, [transcript]);

  const handleLineChange = (lineId: string, field: keyof DialogueLine, value: any) => {
    const updatedLines = lines.map(line => 
      line.id === lineId ? { ...line, [field]: value } : line
    );
    setLines(updatedLines);
    onChange({ ...transcript, lines: updatedLines });
  };

  const addLine = () => {
    const newLine: DialogueLine = {
      id: `line-${Date.now()}`,
      characterId: characters[0]?.id || '',
      text: 'New dialogue line...',
      emotion: 'neutral',
      timing: { start: 0, duration: 3 }
    };
    const updatedLines = [...lines, newLine];
    setLines(updatedLines);
    onChange({ ...transcript, lines: updatedLines });
  };

  const deleteLine = (lineId: string) => {
    const updatedLines = lines.filter(line => line.id !== lineId);
    setLines(updatedLines);
    onChange({ ...transcript, lines: updatedLines });
  };

  const startEditing = (line: DialogueLine) => {
    setEditingId(line.id);
    setEditedText(line.text);
  };

  const saveEdit = (lineId: string) => {
    handleLineChange(lineId, 'text', editedText);
    setEditingId(null);
    setEditedText('');
  };

  const generateAudioForLine = async (line: DialogueLine) => {
    const character = characters.find(c => c.id === line.characterId);
    if (!character) return;

    try {
      const utterance = new SpeechSynthesisUtterance(line.text);
      const voices = speechSynthesis.getVoices();
      const voice = voices.find(v => v.name === character.voiceSettings.voiceId) || voices[0];
      
      utterance.voice = voice;
      utterance.pitch = character.voiceSettings.pitch;
      utterance.rate = character.voiceSettings.rate;
      utterance.volume = character.voiceSettings.volume;
      
      speechSynthesis.speak(utterance);
    } catch (err) {
      console.error('Audio generation failed:', err);
    }
  };

  return (
    <div className="space-y-4">
      <div className="flex justify-between items-center">
        <h4 className="font-medium">Dialogue Lines ({lines.length})</h4>
        <button
          onClick={addLine}
          className="px-3 py-1 bg-green-600 hover:bg-green-700 text-white rounded text-sm transition-colors"
        >
          + Add Line
        </button>
      </div>

      <div className="space-y-3 max-h-96 overflow-y-auto">
        {lines.length === 0 ? (
          <p className="text-gray-400 text-sm">No dialogue lines. Generate or add manually.</p>
        ) : (
          lines.map((line, index) => {
            const character = characters.find(c => c.id === line.characterId);
            
            return (
              <div key={line.id} className="bg-gray-900 p-3 rounded-lg border border-gray-700">
                <div className="flex justify-between items-start mb-2">
                  <select
                    value={line.characterId}
                    onChange={e => handleLineChange(line.id, 'characterId', e.target.value)}
                    className="text-sm bg-gray-800 border border-gray-600 rounded px-2 py-1"
                  >
                    {characters.map(char => (
                      <option key={char.id} value={char.id}>{char.name}</option>
                    ))}
                  </select>
                  
                  <div className="flex gap-1">
                    <button
                      onClick={() => generateAudioForLine(line)}
                      className="text-xs px-2 py-1 bg-purple-600 hover:bg-purple-700 text-white rounded transition-colors"
                    >
                      🔊
                    </button>
                    <button
                      onClick={() => deleteLine(line.id)}
                      className="text-xs px-2 py-1 bg-red-600 hover:bg-red-700 text-white rounded transition-colors"
                    >
                      🗑️
                    </button>
                  </div>
                </div>

                {editingId === line.id ? (
                  <div className="space-y-2">
                    <textarea
                      value={editedText}
                      onChange={e => setEditedText(e.target.value)}
                      className="w-full px-2 py-1 bg-gray-800 border border-gray-600 rounded text-sm"
                      rows={2}
                      autoFocus
                    />
                    <div className="flex gap-2">
                      <button
                        onClick={() => saveEdit(line.id)}
                        className="text-xs px-2 py-1 bg-blue-600 hover:bg-blue-700 text-white rounded transition-colors"
                      >
                        Save
                      </button>
                      <button
                        onClick={() => setEditingId(null)}
                        className="text-xs px-2 py-1 bg-gray-600 hover:bg-gray-500 text-white rounded transition-colors"
                      >
                        Cancel
                      </button>
                    </div>
                  </div>
                ) : (
                  <div 
                    onClick={() => startEditing(line)}
                    className="cursor-pointer hover:bg-gray-800 p-2 rounded"
                  >
                    <p className="text-sm font-medium">{character?.name}</p>
                    <p className="text-sm text-gray-300">"{line.text}"</p>
                    {line.emotion && (
                      <p className="text-xs text-gray-400 mt-1">Emotion: {line.emotion}</p>
                    )}
                    {line.timing && (
                      <p className="text-xs text-gray-400">
                        Timing: {line.timing.start}s - {(line.timing.start + line.timing.duration).toFixed(1)}s
                      </p>
                    )}
                  </div>
                )}
              </div>
            );
          })
        )}
      </div>

      <div className="flex gap-2 pt-4 border-t border-gray-700">
        <button
          onClick={() => {
            const blob = new Blob([JSON.stringify(transcript, null, 2)], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `transcript-${transcript.sceneId}.json`;
            a.click();
            URL.revokeObjectURL(url);
          }}
          className="px-3 py-1 bg-blue-600 hover:bg-blue-700 text-white rounded text-sm transition-colors"
        >
          📥 Export
        </button>
        <button
          onClick={() => {
            // Generate full audio
          }}
          className="px-3 py-1 bg-purple-600 hover:bg-purple-700 text-white rounded text-sm transition-colors"
        >
          🎬 Generate Full Audio
        </button>
      </div>
    </div>
  );
};

STEP 17: Create Settings & Configuration Components
-------------------------------------------
Create: src/components/settings/SettingsPanel.tsx
Content:
import { useState, useEffect } from 'react';
import { GenerationSettings } from '../../types';
import { HARDWARE_PRESETS, DEFAULT_SETTINGS, TTS_ENGINES } from '../../constants';
import { storage } from '../../utils/storage';

interface SettingsPanelProps {
  settings: GenerationSettings;
  onSettingsChange: (settings: GenerationSettings) => void;
  systemInfo: any;
}

export const SettingsPanel = ({ settings, onSettingsChange, systemInfo }: SettingsPanelProps) => {
  const [localSettings, setLocalSettings] = useState<GenerationSettings>(settings);
  const [voices, setVoices] = useState<SpeechSynthesisVoice[]>([]);

  useEffect(() => {
    setLocalSettings(settings);
  }, [settings]);

  useEffect(() => {
    const loadVoices = () => {
      const availableVoices = speechSynthesis.getVoices();
      setVoices(availableVoices);
    };
    
    loadVoices();
    speechSynthesis.onvoiceschanged = loadVoices;
    
    return () => {
      speechSynthesis.onvoiceschanged = null;
    };
  }, []);

  const handleChange = (key: keyof GenerationSettings, value: any) => {
    const updated = { ...localSettings, [key]: value };
    setLocalSettings(updated);
    onSettingsChange(updated);
  };

  const applyPreset = (presetKey: string) => {
    const preset = HARDWARE_PRESETS[presetKey as keyof typeof HARDWARE_PRESETS];
    if (preset) {
      const updated = {
        ...localSettings,
        resolution: preset.resolution,
        steps: preset.steps,
        fps: preset.fps
      };
      setLocalSettings(updated);
      onSettingsChange(updated);
    }
  };

  const detectVRAM = () => {
    if (systemInfo?.vramTotal) {
      const vramMB = Math.round(systemInfo.vramTotal / 1024 / 1024);
      if (vramMB <= 6144) {
        applyPreset('fast');
      } else if (vramMB <= 8192) {
        applyPreset('balanced');
      } else {
        applyPreset('quality');
      }
    }
  };

  return (
    <div className="space-y-8">
      <div>
        <h3 className="text-xl font-semibold mb-4 flex items-center gap-2">
          🎮 Hardware Optimization
        </h3>
        
        <div className="bg-gray-900 p-4 rounded-lg border border-gray-700 mb-4">
          <div className="grid grid-cols-2 gap-4 text-sm">
            <div>GPU: <span className="font-mono">{systemInfo?.gpuModel || 'Unknown'}</span></div>
            <div>VRAM: <span className="font-mono">
              {systemInfo?.vramTotal ? `${Math.round(systemInfo.vramTotal / 1024 / 1024)} MB` : 'Unknown'}
            </span></div>
            <div>CUDA: <span className="font-mono">{systemInfo?.cudaVersion || 'Unknown'}</span></div>
            <div>PyTorch: <span className="font-mono">{systemInfo?.pytorchVersion || 'Unknown'}</span></div>
          </div>
          <button
            onClick={detectVRAM}
            className="mt-3 px-3 py-1 bg-blue-600 hover:bg-blue-700 text-white rounded text-sm transition-colors"
          >
            🔍 Auto-Detect Optimal Settings
          </button>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 mb-6">
          {Object.entries(HARDWARE_PRESETS).map(([key, preset]) => (
            <button
              key={key}
              onClick={() => applyPreset(key)}
              className="p-4 bg-gray-800 hover:bg-gray-700 border border-gray-700 rounded-lg text-left transition-colors"
            >
              <div className="font-semibold">{preset.name}</div>
              <div className="text-xs text-gray-400 mt-1">
                {preset.resolution.width}x{preset.resolution.height}, {preset.steps} steps, {preset.fps} FPS
              </div>
              <div className="text-xs text-gray-500 mt-1">{preset.description}</div>
            </button>
          ))}
        </div>
      </div>

      <div>
        <h3 className="text-xl font-semibold mb-4">⚙️ Generation Settings</h3>
        
        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          <div>
            <label className="block text-sm font-medium mb-2">
              Resolution: {localSettings.resolution.width} × {localSettings.resolution.height}
            </label>
            <div className="grid grid-cols-2 gap-2">
              <input
                type="number"
                value={localSettings.resolution.width}
                onChange={e => handleChange('resolution', { 
                  ...localSettings.resolution, 
                  width: parseInt(e.target.value) 
                })}
                className="px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
                min="256"
                max="1024"
              />
              <input
                type="number"
                value={localSettings.resolution.height}
                onChange={e => handleChange('resolution', { 
                  ...localSettings.resolution, 
                  height: parseInt(e.target.value) 
                })}
                className="px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
                min="256"
                max="1024"
              />
            </div>
          </div>

          <div>
            <label className="block text-sm font-medium mb-2">
              FPS: {localSettings.fps}
            </label>
            <select
              value={localSettings.fps}
              onChange={e => handleChange('fps', parseInt(e.target.value))}
              className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
            >
              <option value="12">12 FPS (animatic)</option>
              <option value="24">24 FPS (cinema)</option>
              <option value="30">30 FPS (smooth)</option>
            </select>
          </div>

          <div>
            <label className="block text-sm font-medium mb-2">
              Steps: {localSettings.steps}
            </label>
            <input
              type="range"
              min="4"
              max="32"
              value={localSettings.steps}
              onChange={e => handleChange('steps', parseInt(e.target.value))}
              className="w-full"
            />
            <p className="text-xs text-gray-400 mt-1">
              Higher = better quality, slower generation
            </p>
          </div>

          <div>
            <label className="block text-sm font-medium mb-2">
              CFG Scale: {localSettings.cfgScale}
            </label>
            <input
              type="range"
              min="1"
              max="10"
              step="0.5"
              value={localSettings.cfgScale}
              onChange={e => handleChange('cfgScale', parseFloat(e.target.value))}
              className="w-full"
            />
          </div>
        </div>
      </div>

      <div>
        <h3 className="text-xl font-semibold mb-4">🎤 Audio Settings</h3>
        
        <div className="space-y-4">
          <div className="flex items-center gap-3">
            <input
              type="checkbox"
              id="enableAudio"
              checked={localSettings.enableAudio}
              onChange={e => handleChange('enableAudio', e.target.checked)}
              className="rounded"
            />
            <label htmlFor="enableAudio" className="text-sm font-medium">
              Enable Audio Generation
            </label>
          </div>

          {localSettings.enableAudio && (
            <>
              <div>
                <label className="block text-sm font-medium mb-2">TTS Engine</label>
                <select
                  value={localSettings.ttsEngine}
                  onChange={e => handleChange('ttsEngine', e.target.value)}
                  className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
                >
                  {TTS_ENGINES.map(engine => (
                    <option key={engine.id} value={engine.id}>
                      {engine.name} - {engine.description}
                    </option>
                  ))}
                </select>
              </div>

              <div>
                <label className="block text-sm font-medium mb-2">Audio Format</label>
                <select
                  value={localSettings.audioFormat}
                  onChange={e => handleChange('audioFormat', e.target.value)}
                  className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
                >
                  <option value="wav">WAV (Uncompressed)</option>
                  <option value="mp3">MP3 (Compressed)</option>
                  <option value="aac">AAC (High Quality)</option>
                </select>
              </div>

              {localSettings.ttsEngine === 'web-speech' && voices.length > 0 && (
                <div>
                  <label className="block text-sm font-medium mb-2">
                    Available Voices ({voices.length})
                  </label>
                  <select className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white">
                    {voices.map(voice => (
                      <option key={voice.name} value={voice.name}>
                        {voice.name} ({voice.lang})
                      </option>
                    ))}
                  </select>
                  <p className="text-xs text-gray-400 mt-1">
                    These voices will be available for character assignment
                  </p>
                </div>
              )}

              {localSettings.ttsEngine === 'coqui-tts' && (
                <div className="bg-yellow-900 bg-opacity-20 border border-yellow-700 p-3 rounded">
                  <p className="text-sm text-yellow-200">
                    ⚠️ Coqui TTS requires a local server. Run: 
                    <code className="ml-2 px-2 py-1 bg-gray-800 rounded">python -m TTS.server --list_models</code>
                  </p>
                </div>
              )}

              {localSettings.ttsEngine === 'elevenlabs' && (
                <div>
                  <label className="block text-sm font-medium mb-2">API Key</label>
                  <input
                    type="password"
                    className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white"
                    placeholder="sk_xxxxxxxxxxxxxxxxxxxx"
                  />
                  <p className="text-xs text-gray-400 mt-1">
                    Requires ElevenLabs subscription for consistent voices
                  </p>
                </div>
              )}
            </>
          )}
        </div>
      </div>

      <div>
        <h3 className="text-xl font-semibold mb-4">📁 Model Paths</h3>
        
        <div className="space-y-4">
          <div>
            <label className="block text-sm font-medium mb-2">Model Type</label>
            <div className="flex gap-4">
              <label className="flex items-center gap-2">
                <input
                  type="radio"
                  value="gguf"
                  checked={localSettings.modelType === 'gguf'}
                  onChange={() => handleChange('modelType', 'gguf')}
                  className="rounded"
                />
                <span>GGUF (Recommended for 6GB VRAM)</span>
              </label>
              <label className="flex items-center gap-2">
                <input
                  type="radio"
                  value="checkpoint"
                  checked={localSettings.modelType === 'checkpoint'}
                  onChange={() => handleChange('modelType', 'checkpoint')}
                  className="rounded"
                />
                <span>Checkpoint (Full quality)</span>
              </label>
            </div>
          </div>

          {['modelPath', 'vaePath', 'clipPath'].map((field) => (
            <div key={field}>
              <label className="block text-sm font-medium mb-2">
                {field.replace('Path', '').toUpperCase()} Path
              </label>
              <input
                type="text"
                value={localSettings[field as keyof GenerationSettings]}
                onChange={e => handleChange(field as keyof GenerationSettings, e.target.value)}
                className="w-full px-3 py-2 border rounded-lg bg-gray-800 border-gray-700 text-white font-mono text-sm"
              />
            </div>
          ))}
        </div>
      </div>

      <div className="flex gap-3 pt-6">
        <button
          onClick={() => {
            storage.saveSettings(localSettings);
            alert('Settings saved!');
          }}
          className="bg-blue-600 hover:bg-blue-700 text-white py-2 px-6 rounded-lg transition-colors"
        >
          Save Settings
        </button>
        <button
          onClick={() => {
            setLocalSettings(DEFAULT_SETTINGS);
            onSettingsChange(DEFAULT_SETTINGS);
          }}
          className="bg-gray-700 hover:bg-gray-600 text-white py-2 px-6 rounded-lg transition-colors"
        >
          Reset to Defaults
        </button>
      </div>
    </div>
  );
};

STEP 18: Create Generation & Monitoring Components
-------------------------------------------
Create: src/components/generation/GenerationPanel.tsx
Content:
import { useState, useEffect } from 'react';
import { Scene, Character, GenerationSettings } from '../../types';
import { useComfyUI } from '../../hooks/useComfyUI';
import { useLogger } from '../../hooks/useLogger';

interface GenerationPanelProps {
  scenes: Scene[];
  characters: Character[];
  settings: GenerationSettings;
  project: any;
  onSceneComplete: (sceneId: string, videoUrl: string, audioUrl?: string) => void;
}

export const GenerationPanel = ({ 
  scenes, 
  characters, 
  settings, 
  project,
  onSceneComplete 
}: GenerationPanelProps) => {
  const [isGenerating, setIsGenerating] = useState(false);
  const [currentSceneIndex, setCurrentSceneIndex] = useState(0);
  const [progress, setProgress] = useState(0);
  const [status, setStatus] = useState('Idle');
  
  const comfyUI = useComfyUI();
  const { info, error, success } = useLogger();

  const generateSceneAudio = async (scene: Scene, sceneCharacters: Character[]): Promise<string | null> => {
    if (!scene.transcript || !settings.enableAudio) return null;

    try {
      info('audio', `Generating audio for scene: ${scene.title}`);
      setStatus('Generating audio...');

      const audioContext = new AudioContext();
      const destination = audioContext.createMediaStreamDestination();
      const mediaRecorder = new MediaRecorder(destination.stream);
      const audioChunks: BlobPart[] = [];

      mediaRecorder.ondataavailable = event => {
        audioChunks.push(event.data);
      };

      return new Promise((resolve, reject) => {
        mediaRecorder.onstop = () => {
          const audioBlob = new Blob(audioChunks, { type: 'audio/wav' });
          const audioUrl = URL.createObjectURL(audioBlob);
          success('audio', 'Audio generation completed');
          resolve(audioUrl);
        };

        mediaRecorder.onerror = (err) => {
          error('audio', 'Audio generation failed', err);
          reject(err);
        };

        mediaRecorder.start();

        let currentTime = 0;
        const speakNextLine = (index: number) => {
          if (index >= scene.transcript!.lines.length) {
            setTimeout(() => mediaRecorder.stop(), 500);
            return;
          }

          const line = scene.transcript!.lines[index];
          const character = sceneCharacters.find(c => c.id === line.characterId);
          
          if (!character) {
            speakNextLine(index + 1);
            return;
          }

          const utterance = new SpeechSynthesisUtterance(line.text);
          const voices = speechSynthesis.getVoices();
          const voice = voices.find(v => v.name === character.voiceSettings.voiceId) || voices[0];
          
          utterance.voice = voice;
          utterance.pitch = character.voiceSettings.pitch;
          utterance.rate = character.voiceSettings.rate;
          utterance.volume = character.voiceSettings.volume;

          const lineDuration = line.timing?.duration || 3;
          const pauseDuration = line.timing ? line.timing.start - currentTime : 0.5;

          setTimeout(() => {
            speechSynthesis.speak(utterance);
            currentTime += lineDuration;
            
            utterance.onend = () => {
              setTimeout(() => speakNextLine(index + 1), 200);
            };
          }, Math.max(0, pauseDuration * 1000));
        };

        setTimeout(() => speakNextLine(0), 500);
      });

    } catch (err) {
      error('audio', 'Failed to generate scene audio', err);
      return null;
    }
  };

  const generateScene = async (scene: Scene) => {
    try {
      setStatus(`Generating: ${scene.title}`);
      info('generation', `Starting generation for scene: ${scene.title}`);

      let audioUrl: string | null = null;
      if (settings.enableAudio && scene.transcript) {
        const sceneCharacters = characters.filter(c => 
          scene.characters.includes(c.id)
        );
        audioUrl = await generateSceneAudio(scene, sceneCharacters);
      }

      const queueResponse = await comfyUI.queuePrompt(
        scene,
        settings,
        characters,
        project.style,
        settings.enableAudio,
        audioUrl || undefined
      );

      if (!queueResponse) {
        throw new Error('Failed to queue workflow');
      }

      const promptId = queueResponse.prompt_id;
      let completed = false;
      let attempts = 0;
      const maxAttempts = 600;

      while (!completed && attempts < maxAttempts) {
        await new Promise(resolve => setTimeout(resolve, 1000));
        
        const history = await comfyUI.checkProgress(promptId);
        if (history?.status.completed) {
          completed = true;
          
          const outputNode = history.outputs[Object.keys(history.outputs)[0]];
          const videoUrl = outputNode?.video?.[0]?.filename || 
                          outputNode?.images?.[0]?.filename;
          
          if (videoUrl) {
            const fullUrl = `http://127.0.0.1:8188/view?filename=${encodeURIComponent(videoUrl)}`;
            success('generation', 'Scene generated successfully');
            onSceneComplete(scene.id, fullUrl, audioUrl || undefined);
          }
        } else if (history?.status.failed) {
          throw new Error('Generation failed');
        }
        
        attempts++;
        setProgress(Math.min((attempts / maxAttempts) * 100, 95));
      }

      if (!completed) {
        throw new Error('Generation timed out');
      }

    } catch (err) {
      error('generation', `Failed to generate scene: ${scene.title}`, err);
      throw err;
    }
  };

  const generateAll = async () => {
    if (isGenerating) return;
    
    setIsGenerating(true);
    setCurrentSceneIndex(0);
    setProgress(0);
    
    const pendingScenes = scenes.filter(s => s.status !== 'completed');
    
    for (let i = 0; i < pendingScenes.length; i++) {
      setCurrentSceneIndex(i);
      const scene = pendingScenes[i];
      
      try {
        await generateScene(scene);
        setProgress((i + 1) / pendingScenes.length * 100);
      } catch (err) {
        error('generation', `Failed at scene ${i + 1}: ${scene.title}`);
        break;
      }
    }
    
    setIsGenerating(false);
    setStatus('Completed');
    success('generation', `Generated ${pendingScenes.length} scenes`);
  };

  const stopGeneration = async () => {
    await comfyUI.interrupt();
    setIsGenerating(false);
    setStatus('Stopped');
    info('generation', 'Generation stopped by user');
  };

  return (
    <div className="bg-gray-900 p-6 rounded-lg border border-gray-700">
      <div className="flex justify-between items-center mb-4">
        <h2 className="text-2xl font-bold">🎬 Generation Control</h2>
        <div className="text-sm text-gray-400">
          Queue: {comfyUI.queuePosition || 0}
        </div>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mb-6">
        <div className="bg-gray-800 p-4 rounded-lg">
          <h3 className="text-sm font-medium text-gray-400 mb-1">Status</h3>
          <p className="text-lg font-semibold">{status}</p>
        </div>
        
        <div className="bg-gray-800 p-4 rounded-lg">
          <h3 className="text-sm font-medium text-gray-400 mb-1">Progress</h3>
          <p className="text-lg font-semibold">
            {isGenerating ? `${Math.round(progress)}%` : '0%'}
          </p>
          <div className="w-full bg-gray-700 rounded-full h-2 mt-2 progress-bar">
            <div 
              className="bg-blue-600 h-2 rounded-full transition-all duration-300"
              style={{ width: `${progress}%` }}
            />
          </div>
        </div>
        
        <div className="bg-gray-800 p-4 rounded-lg">
          <h3 className="text-sm font-medium text-gray-400 mb-1">Current Scene</h3>
          <p className="text-lg font-semibold truncate">
            {isGenerating && scenes[currentSceneIndex]?.title || 'None'}
          </p>
        </div>
      </div>

      <div className="flex gap-4 mb-6">
        <button
          onClick={generateAll}
          disabled={isGenerating || scenes.length === 0}
          className="flex-1 bg-green-600 hover:bg-green-700 disabled:bg-gray-700 disabled:opacity-50 text-white py-3 px-6 rounded-lg transition-colors font-medium"
        >
          {isGenerating ? 'Generating...' : `🎬 Generate All (${scenes.filter(s => s.status !== 'completed').length} pending)`}
        </button>
        
        {isGenerating && (
          <button
            onClick={stopGeneration}
            className="bg-red-600 hover:bg-red-700 text-white py-3 px-6 rounded-lg transition-colors font-medium"
          >
            ⏹️ Stop
          </button>
        )}
      </div>

      <div className="bg-gray-800 p-4 rounded-lg">
        <h3 className="font-medium mb-3">Scene Queue</h3>
        <div className="space-y-2 max-h-64 overflow-y-auto">
          {scenes.map((scene, index) => (
            <div 
              key={scene.id}
              className={`flex justify-between items-center p-2 rounded ${
                index === currentSceneIndex && isGenerating ? 'bg-blue-900 bg-opacity-30' : 'bg-gray-900'
              }`}
            >
              <div>
                <p className="font-medium text-sm">{scene.title}</p>
                <p className="text-xs text-gray-400">
                  {scene.duration}s • {scene.characters.length} characters
                  {scene.transcript && ` • ${scene.transcript.lines.length} lines`}
                </p>
              </div>
              <div className="flex items-center gap-2">
                {scene.transcript && (
                  <span className="text-xs px-2 py-1 bg-purple-700 bg-opacity-50 rounded">📝</span>
                )}
                {settings.enableAudio && (
                  <span className="text-xs px-2 py-1 bg-green-700 bg-opacity-50 rounded">🎤</span>
                )}
                <span className={`text-xs px-2 py-1 rounded ${
                  scene.status === 'completed' ? 'bg-green-700' : 
                  scene.status === 'generating' ? 'bg-yellow-700' : 'bg-gray-700'
                }`}>
                  {scene.status}
                </span>
              </div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
};

STEP 19: Create Guide & Documentation Components
-------------------------------------------
Create: src/components/guide/SetupGuide.tsx
Content:
export const SetupGuide = () => {
  const sections = [
    {
      title: '🚀 Quick Start',
      content: `
1. **Start ComfyUI**: 
   \`\`\`bash
   cd C:\\ComfyUI
   venv\\Scripts\\activate
   python main.py --lowvram --listen
   \`\`\`

2. **Start Ollama**:
   \`\`\`bash
   ollama serve
   ollama pull llama3.2
   \`\`\`

3. **Open LTX-2 Anime Agent**:
   - Open dist/index.html in your browser
   - Or serve via HTTP: npx serve dist

4. **Verify Connections**:
   - Check ComfyUI status (should be green)
   - Check Ollama status (should be green)
   - GPU info should display your GTX 1660 Ti
      `
    },
    {
      title: '🎬 Creating Your First Video',
      content: `
1. **Create Project**: Click "New Project" → Enter details → Choose style
2. **Generate Characters**: Click "🤖 AI Generate" in Characters tab
3. **Create Episode**: Click "🤖 AI Story" in Episodes tab
4. **Generate Transcript**: Click "🤖 AI Generate" in any scene
5. **Generate Video**: Click "🎬 Generate All" in Generation tab

**For GTX 1660 Ti**: Use "Fast" preset (512x384, 8 steps) for best results.
      `
    },
    {
      title: '🔊 Audio & Voice Setup',
      content: `
**Web Speech API** (Default, Free):
- Works offline in modern browsers
- Voice consistency depends on browser
- Click "🔄 Load Voices" in character voice settings

**Coqui TTS** (Recommended for Quality):
\`\`\`bash
pip install TTS
python -m TTS.server --list_models
python -m TTS.server --model_name tts_models/multilingual/multi-dataset/your_tts
\`\`\`

**ElevenLabs** (Best Quality):
- Get API key from elevenlabs.io
- Enter in Settings → Audio → API Key
- Most consistent voices but requires subscription

**Voice Consistency Tips**:
1. Assign each character a unique voice preset
2. Test voices using "🔊 Test Voice" button
3. Adjust pitch/rate to differentiate characters
4. Use same voice settings across all episodes
      `
    },
    {
      title: '💾 Project Management',
      content: `
**Auto-Save**: Projects are saved to browser localStorage automatically

**Export Project**:
- Project data is saved as JSON
- Includes all characters, episodes, scenes, transcripts
- Click "Export Project" in dashboard

**Import Project**:
- Click "Import Project" in dashboard
- Select previously exported JSON file

**Backup**: 
- Export project before major changes
- localStorage is cleared if you clear browser data
      `
    },
    {
      title: '🐛 Troubleshooting',
      content: `
**"CUDA Out of Memory"**:
- Lower resolution to 512x384
- Reduce steps to 6-8
- Restart ComfyUI with --lowvram flag
- Close other GPU applications

**"Black/Corrupted Video"**:
- Update ComfyUI-LTXVideo: cd custom_nodes/ComfyUI-LTXVideo && git pull
- Verify VAE file is correct
- Try different sampler (euler → dpmpp_2m)

**"No Audio Generated"**:
- Enable audio in Settings → Audio
- Check browser permissions for microphone (for recording)
- Verify transcript has dialogue lines
- For Coqui TTS: ensure server is running

**"Voice Not Consistent"**:
- Use same voice ID across all scenes
- Save character settings
- For Web Speech API: voices may vary by browser/OS
- Consider Coqui TTS or ElevenLabs for consistency

**"Ollama Not Generating"**:
- Check Ollama is running: ollama list
- Pull a model: ollama pull llama3.2
- Try smaller model: ollama pull gemma2:2b
- Check Ollama URL in settings

**"Generation Very Slow"**:
- Use GGUF Q4_K_M model (1.4GB)
- Reduce steps to 8
- Lower resolution to 512x384
- Ensure --lowvram flag is used
- Close Chrome/browsers during generation
      `
    },
    {
      title: '🎮 GTX 1660 Ti Optimization',
      content: `
**Recommended Settings**:
- **Resolution**: 512×384 (Fast) or 640×480 (Quality)
- **Steps**: 8 (preview) or 16-24 (final)
- **FPS**: 24 (standard) or 30 (smooth)
- **Model**: ltx-video-2b-Q4_K_M.gguf (1.4GB)
- **Launch**: python main.py --lowvram --listen

**Performance Expectations**:
- **Fast Preset**: 1-2 minutes per 8-second scene
- **Balanced Preset**: 2-4 minutes per scene
- **Quality Preset**: 3-6 minutes per scene

**VRAM Management**:
- Always use --lowvram flag
- Generate scenes sequentially (not parallel)
- Monitor GPU temp (keep under 80°C)
- Restart ComfyUI between large batches

**Speed Optimizations**:
- Use SSD for model storage
- Disable Windows visual effects
- Update GPU drivers to latest
- Close all unnecessary applications
- Use 512×384 for iterative testing
      `
    }
  ];

  return (
    <div className="max-w-4xl mx-auto space-y-8">
      <div className="text-center mb-8">
        <h1 className="text-4xl font-bold mb-4">📖 LTX-2 Anime Agent Guide</h1>
        <p className="text-gray-400">
          Complete guide to creating anime videos with AI
        </p>
      </div>

      {sections.map((section, index) => (
        <div key={index} className="bg-gray-900 p-6 rounded-lg border border-gray-700">
          <h2 className="text-2xl font-bold mb-4">{section.title}</h2>
          <div className="prose prose-invert max-w-none">
            <pre className="bg-gray-800 p-4 rounded-lg overflow-x-auto text-sm">
              {section.content}
            </pre>
          </div>
        </div>
      ))}

      <div className="bg-blue-900 bg-opacity-20 border border-blue-700 p-6 rounded-lg">
        <h2 className="text-2xl font-bold mb-4">📞 Need More Help?</h2>
        <div className="space-y-2 text-sm">
          <p>• Check the <strong>Log Panel</strong> for detailed error messages</p>
          <p>• Export logs and share for troubleshooting</p>
          <p>• Visit ComfyUI GitHub: github.com/comfyanonymous/ComfyUI</p>
          <p>• LTX-Video GitHub: github.com/Lightricks/LTX-Video</p>
          <p>• Ollama Documentation: ollama.ai</p>
        </div>
      </div>
    </div>
  );
};

STEP 20: Create Logs & Monitoring Panel
-------------------------------------------
Create: src/components/logs/LogPanel.tsx
Content:
import { useState } from 'react';
import { useLogger, LogEntry, LogLevel, LogCategory } from '../../hooks/useLogger';

const levelColors: Record<LogLevel, string> = {
  info: 'text-blue-400',
  success: 'text-green-400',
  warning: 'text-yellow-400',
  error: 'text-red-400',
  debug: 'text-gray-400'
};

const categoryIcons: Record<LogCategory, string> = {
  system: '⚙️',
  comfyui: '🎨',
  ollama: '🤖',
  generation: '🎬',
  ai: '🧠',
  export: '📤',
  audio: '🎤'
};

export const LogPanel = () => {
  const { logs, clear, exportLogs, filterLogs } = useLogger();
  const [selectedLevel, setSelectedLevel] = useState<LogLevel | undefined>();
  const [selectedCategory, setSelectedCategory] = useState<LogCategory | undefined>();
  const [showDetails, setShowDetails] = useState<Record<string, boolean>>({});

  const filteredLogs = filterLogs(selectedLevel, selectedCategory);

  const toggleDetails = (logId: string) => {
    setShowDetails(prev => ({ ...prev, [logId]: !prev[logId] }));
  };

  return (
    <div className="bg-gray-900 p-6 rounded-lg border border-gray-700">
      <div className="flex justify-between items-center mb-4">
        <h2 className="text-2xl font-bold">📊 Logs & Monitoring</h2>
        <div className="flex gap-2">
          <button
            onClick={clear}
            className="px-3 py-1 bg-gray-700 hover:bg-gray-600 text-white rounded transition-colors text-sm"
          >
            Clear
          </button>
          <button
            onClick={exportLogs}
            className="px-3 py-1 bg-blue-600 hover:bg-blue-700 text-white rounded transition-colors text-sm"
          >
            Export
          </button>
        </div>
      </div>

      <div className="flex gap-4 mb-4 flex-wrap">
        <select
          value={selectedLevel || ''}
          onChange={e => setSelectedLevel(e.target.value as LogLevel || undefined)}
          className="px-3 py-1 bg-gray-800 border border-gray-700 rounded text-sm"
        >
          <option value="">All Levels</option>
          <option value="info">Info</option>
          <option value="success">Success</option>
          <option value="warning">Warning</option>
          <option value="error">Error</option>
          <option value="debug">Debug</option>
        </select>

        <select
          value={selectedCategory || ''}
          onChange={e => setSelectedCategory(e.target.value as LogCategory || undefined)}
          className="px-3 py-1 bg-gray-800 border border-gray-700 rounded text-sm"
        >
          <option value="">All Categories</option>
          <option value="system">System</option>
          <option value="comfyui">ComfyUI</option>
          <option value="ollama">Ollama</option>
          <option value="generation">Generation</option>
          <option value="ai">AI</option>
          <option value="export">Export</option>
          <option value="audio">Audio</option>
        </select>

        <div className="text-sm text-gray-400 self-center">
          Showing {filteredLogs.length} of {logs.length} logs
        </div>
      </div>

      <div className="space-y-2 max-h-96 overflow-y-auto font-mono text-xs custom-scrollbar">
        {filteredLogs.length === 0 ? (
          <p className="text-gray-500">No logs to display</p>
        ) : (
          filteredLogs.map(log => (
            <div key={log.id} className="bg-gray-800 p-3 rounded border border-gray-700 hover:border-gray-600 transition-colors">
              <div 
                className="flex justify-between items-start cursor-pointer"
                onClick={() => toggleDetails(log.id)}
              >
                <div className="flex-1">
                  <div className="flex items-center gap-2">
                    <span className="text-gray-400">
                      {log.timestamp.toLocaleTimeString()}
                    </span>
                    <span className={levelColors[log.level]}>
                      [{log.level.toUpperCase()}]
                    </span>
                    <span className="text-gray-500">
                      {categoryIcons[log.category]} {log.category.toUpperCase()}
                    </span>
                  </div>
                  <p className="text-white mt-1">{log.message}</p>
                </div>
                {(log.details || log.solution) && (
                  <span className="text-gray-500">
                    {showDetails[log.id] ? '▼' : '▶'}
                  </span>
                )}
              </div>

              {showDetails[log.id] && (log.details || log.solution) && (
                <div className="mt-2 pt-2 border-t border-gray-700 space-y-2">
                  {log.details && (
                    <div>
                      <p className="text-gray-400 text-xs mb-1">Details:</p>
                      <pre className="text-gray-300 whitespace-pre-wrap">{log.details}</pre>
                    </div>
                  )}
                  {log.solution && (
                    <div>
                      <p className="text-green-400 text-xs mb-1">Solution:</p>
                      <pre className="text-green-300 whitespace-pre-wrap">{log.solution}</pre>
                    </div>
                  )}
                </div>
              )}
            </div>
          ))
        )}
      </div>
    </div>
  );
};

===============================================================================

PHASE 6: MAIN APPLICATION ASSEMBLY
===============================================================================

STEP 21: Create Main App Component
-------------------------------------------
Create: src/App.tsx
Content:
import { useState, useEffect } from 'react';
import { Header } from './components/layout/Header';
import { Tabs } from './components/layout/Tabs';
import { StatusBar } from './components/layout/StatusBar';
import { Dashboard } from './components/projects/Dashboard';
import { CharacterManager } from './components/characters/CharacterManager';
import { EpisodeManager } from './components/episodes/EpisodeManager';
import { GenerationPanel } from './components/generation/GenerationPanel';
import { SettingsPanel } from './components/settings/SettingsPanel';
import { LogPanel } from './components/logs/LogPanel';
import { SetupGuide } from './components/guide/SetupGuide';
import { useLogger } from './hooks/useLogger';
import { useComfyUI } from './hooks/useComfyUI';
import { useOllama } from './hooks/useOllama';
import { storage } from './utils/storage';
import { DEFAULT_SETTINGS } from './constants';
import type { Project, Character, Episode, Scene, GenerationSettings } from './types';

type TabKey = 'dashboard' | 'characters' | 'episodes' | 'generation' | 'settings' | 'logs' | 'guide';

function App() {
  const [activeTab, setActiveTab] = useState<TabKey>('dashboard');
  const [projects, setProjects] = useState<Project[]>([]);
  const [characters, setCharacters] = useState<Character[]>([]);
  const [episodes, setEpisodes] = useState<Episode[]>([]);
  const [currentProject, setCurrentProject] = useState<Project | null>(null);
  const [settings, setSettings] = useState<GenerationSettings>(DEFAULT_SETTINGS);
  
  const logger = useLogger();
  const comfyUI = useComfyUI();
  const ollama = useOllama();

  useEffect(() => {
    const loadedProjects = storage.loadProjects();
    setProjects(loadedProjects);
    
    const loadedSettings = storage.loadSettings();
    if (loadedSettings) {
      setSettings(loadedSettings);
    }

    comfyUI.checkConnection();
    ollama.checkConnection();

    if (typeof speechSynthesis !== 'undefined') {
      speechSynthesis.getVoices();
    }

    logger.info('system', 'LTX-2 Anime Agent initialized');
  }, []);

  useEffect(() => {
    storage.saveProjects(projects);
  }, [projects]);

  useEffect(() => {
    storage.saveSettings(settings);
  }, [settings]);

  useEffect(() => {
    if (currentProject) {
      const projectChars = characters.filter(c => c.projectId === currentProject.id);
      const projectEpisodes = episodes.filter(e => e.projectId === currentProject.id);
      logger.debug('system', `Auto-saved project: ${currentProject.name}`, 
        `${projectChars.length} characters, ${projectEpisodes.length} episodes`);
    }
  }, [characters, episodes, currentProject]);

  const selectProject = (project: Project) => {
    setCurrentProject(project);
    const projectChars = characters.filter(c => c.projectId === project.id);
    const projectEpisodes = episodes.filter(e => e.projectId === project.id);
    
    logger.info('system', `Selected project: ${project.name}`, 
      `${projectChars.length} characters, ${projectEpisodes.length} episodes`);
  };

  const createProject = (projectData: Partial<Project>) => {
    const newProject: Project = {
      ...projectData,
      id: Date.now().toString(),
      created: new Date().toISOString(),
      modified: new Date().toISOString()
    } as Project;
    
    setProjects(prev => [...prev, newProject]);
    setCurrentProject(newProject);
    logger.success('system', 'Project created', newProject.name);
  };

  const deleteProject = (projectId: string) => {
    setProjects(prev => prev.filter(p => p.id !== projectId));
    setCharacters(prev => prev.filter(c => c.projectId !== projectId));
    setEpisodes(prev => prev.filter(e => e.projectId !== projectId));
    
    if (currentProject?.id === projectId) {
      setCurrentProject(null);
    }
    
    logger.warning('system', 'Project deleted', projectId);
  };

  const createCharacter = (charData: Partial<Character>) => {
    if (!currentProject) {
      logger.error('system', 'No project selected');
      return;
    }

    const newCharacter: Character = {
      ...charData,
      id: Date.now().toString(),
      projectId: currentProject.id,
      created: new Date().toISOString()
    } as Character;
    
    setCharacters(prev => [...prev, newCharacter]);
    logger.success('ai', 'Character created', newCharacter.name);
  };

  const updateCharacter = (characterId: string, updates: Partial<Character>) => {
    setCharacters(prev => prev.map(c => 
      c.id === characterId ? { ...c, ...updates } : c
    ));
    logger.info('ai', 'Character updated', characterId);
  };

  const deleteCharacter = (characterId: string) => {
    setCharacters(prev => prev.filter(c => c.id !== characterId));
    logger.warning('ai', 'Character deleted', characterId);
  };

  const generateCharacters = async () => {
    if (!currentProject) {
      logger.error('ai', 'No project selected');
      return;
    }

    const newCharacters = await ollama.generateCharacters(currentProject, 4);
    newCharacters.forEach(char => createCharacter(char));
  };

  const createEpisode = (episodeData: Partial<Episode>) => {
    if (!currentProject) {
      logger.error('system', 'No project selected');
      return;
    }

    const newEpisode: Episode = {
      ...episodeData,
      id: Date.now().toString(),
      projectId: currentProject.id,
      scenes: episodeData.scenes || [],
      created: new Date().toISOString()
    } as Episode;
    
    setEpisodes(prev => [...prev, newEpisode]);
    logger.success('system', 'Episode created', newEpisode.title);
  };

  const updateEpisode = (episodeId: string, updates: Partial<Episode>) => {
    setEpisodes(prev => prev.map(e => 
      e.id === episodeId ? { ...e, ...updates } : e
    ));
    logger.info('system', 'Episode updated', episodeId);
  };

  const generateStory = async () => {
    if (!currentProject) {
      logger.error('ai', 'No project selected');
      return;
    }

    const projectChars = characters.filter(c => c.projectId === currentProject.id);
    const newEpisode = await ollama.generateStory(currentProject, projectChars);
    createEpisode(newEpisode);
  };

  const generateTranscript = async (scene: Scene) => {
    if (!currentProject) {
      logger.error('ai', 'No project selected');
      return;
    }

    const projectChars = characters.filter(c => c.projectId === currentProject.id);
    const transcript = await ollama.generateTranscript(scene, projectChars, currentProject.style);
    
    if (transcript) {
      const episode = episodes.find(e => e.scenes.some(s => s.id === scene.id));
      if (episode) {
        const updatedScenes = episode.scenes.map(s => 
          s.id === scene.id ? { ...s, transcript } : s
        );
        updateEpisode(episode.id, { scenes: updatedScenes });
        logger.success('ai', 'Transcript generated', `${transcript.lines.length} lines`);
      }
    }
  };

  const updateSceneStatus = (sceneId: string, videoUrl: string, audioUrl?: string) => {
    setEpisodes(prev => prev.map(episode => ({
      ...episode,
      scenes: episode.scenes.map(scene => 
        scene.id === sceneId 
          ? { ...scene, status: 'completed' as const, generatedVideoUrl: videoUrl, generatedAudioUrl: audioUrl }
          : scene
      )
    })));
    logger.success('generation', 'Scene completed', sceneId);
  };

  const currentProjectCharacters = currentProject ? 
    characters.filter(c => c.projectId === currentProject.id) : [];
  
  const currentProjectEpisodes = currentProject ? 
    episodes.filter(e => e.projectId === currentProject.id) : [];

  const currentScenes = currentProjectEpisodes.length > 0 ? 
    currentProjectEpisodes[0].scenes : [];

  const tabs: { key: TabKey; label: string; icon: string }[] = [
    { key: 'dashboard', label: 'Dashboard', icon: '🏠' },
    { key: 'characters', label: 'Characters', icon: '👥' },
    { key: 'episodes', label: 'Episodes', icon: '🎬' },
    { key: 'generation', label: 'Generation', icon: '⚡' },
    { key: 'settings', label: 'Settings', icon: '⚙️' },
    { key: 'logs', label: 'Logs', icon: '📊' },
    { key: 'guide', label: 'Guide', icon: '📖' }
  ];

  return (
    <div className="min-h-screen bg-gray-950 text-white">
      <Header 
        title="LTX-2 Anime Agent"
        subtitle="AI-Powered Anime Video Production"
      />
      
      <Tabs 
        tabs={tabs}
        activeTab={activeTab}
        onTabChange={setActiveTab}
      />

      <main className="container mx-auto px-4 py-6 max-w-7xl">
        {activeTab === 'dashboard' && (
          <Dashboard
            projects={projects}
            currentProject={currentProject}
            onSelectProject={selectProject}
            onCreateProject={createProject}
            onDeleteProject={deleteProject}
          />
        )}

        {activeTab === 'characters' && currentProject && (
          <CharacterManager
            project={currentProject}
            characters={currentProjectCharacters}
            onCreateCharacter={createCharacter}
            onUpdateCharacter={updateCharacter}
            onDeleteCharacter={deleteCharacter}
            onGenerateCharacters={generateCharacters}
          />
        )}

        {activeTab === 'episodes' && currentProject && (
          <EpisodeManager
            project={currentProject}
            episodes={currentProjectEpisodes}
            characters={currentProjectCharacters}
            onCreateEpisode={createEpisode}
            onUpdateEpisode={updateEpisode}
            onGenerateStory={generateStory}
            onGenerateTranscript={generateTranscript}
          />
        )}

        {activeTab === 'generation' && currentProject && (
          <GenerationPanel
            scenes={currentScenes}
            characters={currentProjectCharacters}
            settings={settings}
            project={currentProject}
            onSceneComplete={updateSceneStatus}
          />
        )}

        {activeTab === 'settings' && (
          <SettingsPanel
            settings={settings}
            onSettingsChange={setSettings}
            systemInfo={comfyUI.systemInfo}
          />
        )}

        {activeTab === 'logs' && <LogPanel />}

        {activeTab === 'guide' && <SetupGuide />}
      </main>

      <StatusBar
        comfyUIConnected={comfyUI.connected}
        ollamaConnected={ollama.connected}
        currentProject={currentProject}
      />
    </div>
  );
}

export default App;

===============================================================================

PHASE 7: FINAL STYLING & POLISH
===============================================================================

STEP 22: Add Remaining CSS
-------------------------------------------
Edit: src/index.css
Add to end of file:
/* Button shine effect */
.anime-button {
  @apply relative overflow-hidden transition-all duration-300;
}

.anime-button::after {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: linear-gradient(45deg, transparent, rgba(255, 255, 255, 0.1), transparent);
  transform: rotate(30deg);
  transition: all 0.6s;
}

.anime-button:hover::after {
  transform: translateX(100%) rotate(30deg);
}

/* Loading spinner */
.anime-spinner {
  @apply inline-block w-4 h-4 border-2 border-white border-t-transparent rounded-full;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* Status indicators */
.status-connected {
  @apply bg-green-500 w-3 h-3 rounded-full;
  animation: pulse-green 2s infinite;
}

.status-disconnected {
  @apply bg-red-500 w-3 h-3 rounded-full;
}

.status-checking {
  @apply bg-yellow-500 w-3 h-3 rounded-full;
  animation: pulse-yellow 1.5s infinite;
}

@keyframes pulse-green {
  0% { box-shadow: 0 0 0 0 rgba(34, 197, 94, 0.7); }
  70% { box-shadow: 0 0 0 10px rgba(34, 197, 94, 0); }
  100% { box-shadow: 0 0 0 0 rgba(34, 197, 94, 0); }
}

@keyframes pulse-yellow {
  0% { box-shadow: 0 0 0 0 rgba(251, 191, 36, 0.7); }
  70% { box-shadow: 0 0 0 10px rgba(251, 191, 36, 0); }
  100% { box-shadow: 0 0 0 0 rgba(251, 191, 36, 0); }
}

/* Tab animations */
.tab-enter {
  animation: tab-slide-in 0.3s ease-out;
}

@keyframes tab-slide-in {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Voice test button animation */
.voice-test-btn {
  @apply relative;
}

.voice-test-btn.speaking {
  animation: voice-pulse 0.5s infinite;
}

@keyframes voice-pulse {
  0% { box-shadow: 0 0 0 0 rgba(168, 85, 247, 0.7); }
  70% { box-shadow: 0 0 0 10px rgba(168, 85, 247, 0); }
  100% { box-shadow: 0 0 0 0 rgba(168, 85, 247, 0); }
}

/* Responsive design */
@media (max-width: 768px) {
  .anime-card:hover {
    @apply transform-none shadow-none;
  }
  
  .tab-grid {
    @apply grid-cols-2;
  }
}

@media (min-width: 769px) and (max-width: 1024px) {
  .tab-grid {
    @apply grid-cols-3;
  }
}

@media (min-width: 1025px) {
  .tab-grid {
    @apply grid-cols-7;
  }
}

STEP 23: Add Missing Imports
-------------------------------------------
Add to top of src/utils/workflow.ts:
import { ANIMATION_STYLES, MOODS } from '../constants';

===============================================================================

PHASE 8: TESTING & BUILD
===============================================================================

STEP 24: Create Test Files
-------------------------------------------
Create: src/tests/integration.test.ts
Content:
// Integration test outline
describe('LTX-2 Anime Agent Integration', () => {
  test('Project creation and persistence', () => {
    // Test creating a project
    // Test saving to localStorage
    // Test loading from localStorage
  });

  test('Character generation with voice settings', () => {
    // Test AI character generation
    // Test voice preset assignment
    // Test voice settings storage
  });

  test('Transcript generation', () => {
    // Test AI transcript generation
    // Test dialogue line parsing
    // Test timing calculation
  });

  test('Audio generation workflow', () => {
    // Test Web Speech API integration
    // Test audio blob creation
    // Test audio-video merging
  });

  test('ComfyUI workflow generation with audio', () => {
    // Test workflow includes audio nodes
    // Test correct node connections
    // Test fallback for missing VHS
  });

  test('Error handling and logging', () => {
    // Test ComfyUI connection errors
    // Test Ollama errors
    // Test audio generation errors
    // Test log export
  });
});

STEP 25: Build and Verify
-------------------------------------------
Commands:
npm install
npm run build
npm run preview

Final Verification Checklist:
✅ Projects create/save/load
✅ Characters generate with voice settings
✅ Transcripts generate with AI
✅ Audio generation works (Web Speech API)
✅ ComfyUI workflow includes audio nodes
✅ Logs display all categories including audio
✅ Settings persist (including audio settings)
✅ UI is responsive on mobile/tablet
✅ All tabs function correctly
✅ Build completes without errors

===============================================================================

🎉 IMPLEMENTATION COMPLETE!

Total Files Created: 35+
Key Features Added:
- Voice Settings Panel for each character
- AI Transcript Generation integrated with Ollama
- Web Speech API TTS for audio generation
- Audio-Video Workflow merging in ComfyUI
- Voice Consistency across all scenes/episodes
- Real-time Audio Generation progress tracking
- Voice Presets for quick character voice assignment
- Audio Testing per character
- Transcript Editor with manual line editing
- Audio Export capabilities

To Start Development:
1. Copy this entire plan to a file (plan.txt)
2. Run: npm create vite@latest ltx-anime-agent -- --template react-ts
3. Follow each step sequentially
4. Test thoroughly before production build

For GTX 1660 Ti Optimization:
- Default to 512x384 resolution, 8 steps, 24 FPS
- Use GGUF Q4_K_M model (1.4GB)
- Always launch ComfyUI with --lowvram flag
- Generate scenes sequentially to avoid OOM

===============================================================================
