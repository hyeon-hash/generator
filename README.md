import React, { useState, useRef, useEffect, useCallback } from 'react';
import { Upload, Download, Grid, RefreshCcw, LayoutGrid, Shuffle, Move, RotateCw, Layers, Trash2, Heart, Star, Type, Triangle, Sparkles, Wand2, Loader2, AlertCircle, CheckCircle2, Image as ImageIcon, Paintbrush, Plus, X, Code, Copy, Check, Palette, Percent, Monitor, FileImage, ZoomIn, ZoomOut, FileCode, Maximize, Scan, Share2, Link as LinkIcon, Box, Wind, PlayCircle, StopCircle, Video, Square, Activity, Diamond, Eye, Edit3, Globe } from 'lucide-react';
import { initializeApp } from "firebase/app";
import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "firebase/auth";
import { getFirestore, doc, setDoc, getDoc, collection, addDoc } from "firebase/firestore";

// ==================================================================================
// [호스팅 가이드] 
// 이 코드를 외부(Vercel, Netlify 등)에 배포하려면 아래 설정을 본인의 Firebase 프로젝트 설정으로 교체하세요.
// ==================================================================================
const YOUR_FIREBASE_CONFIG = {
  // apiKey: "...",
};
// ==================================================================================

const INITIAL_DEFAULTS = {
  svgContent: `<svg viewBox="0 0 24 24" fill="currentColor" stroke="none" xmlns="http://www.w3.org/2000/svg"><path d="M12 2C7.5 2 4 5 4 9C4 13.5 7 18 12 22C17 18 20 13.5 20 9C20 5 16.5 2 12 2ZM12 4C15.5 4 18 6.5 18 9C18 12.5 15.5 16 12 19C8.5 16 6 12.5 6 9C6 6.5 8.5 4 12 4Z"/></svg>`,
  svgViewBox: "0 0 24 24",
  svgRatio: 1, 
  
  svgContent2: null,
  svgViewBox2: "0 0 24 24",
  svgRatio2: 1,
  hasSvg2: false,
  
  patternType: "grid",
  renderMode: "inline",
  shapeType: "heart",
  shapeText: "♥",
  shapeFillMode: "grid",
  
  textMaskScale: 0.8,
  fitContent: false,

  bgColor: "#f8fafc",
  
  // Style 1 (Main)
  fillColor: "#3b82f6",
  strokeColor: "#000000",
  strokeWidth: 0,
  useOriginalColor: false,
  size: 30,

  // Style 2 (Sub)
  fillColor2: "#ec4899",
  strokeColor2: "#000000",
  strokeWidth2: 0,
  useOriginalColor2: false,
  size2: 30,

  // Mix & Scale Settings
  mixRatio: 50,
  patternScale: 1,

  // 3D & Transform Settings
  tiltX: 0,
  tiltY: 0,
  zSpread: 0,
  zDepthMode: 'random', 
  globalRotation: 0,
  isAutoRotating: false, 

  spacingX: 15,
  spacingY: 15,
  rotation: 0,
  randomRotation: false,
  randomSize: false,
  opacity: 1,
  canvasWidth: 800,
  canvasHeight: 600,
  density: 100
};

// --- Safe Firebase Init ---
let auth, db;
let firebaseReady = false;
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

try {
  let config = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
  if (!config && YOUR_FIREBASE_CONFIG.apiKey) config = YOUR_FIREBASE_CONFIG;

  if (config) {
      const app = initializeApp(config);
      auth = getAuth(app);
      db = getFirestore(app);
      firebaseReady = true;
  }
} catch (e) {
  console.error("Firebase init failed:", e);
}

// --- Clipboard Helper ---
const copyToClipboard = (text) => {
  const textArea = document.createElement("textarea");
  textArea.value = text;
  textArea.style.position = "fixed";
  textArea.style.left = "-9999px";
  document.body.appendChild(textArea);
  textArea.focus();
  textArea.select();
  try {
    document.execCommand('copy');
  } catch (err) {
    console.error('Unable to copy', err);
  }
  document.body.removeChild(textArea);
};

// Toast Component
const Toast = ({ message, type, onClose }) => {
  useEffect(() => {
    const timer = setTimeout(() => { onClose(); }, 3000);
    return () => clearTimeout(timer);
  }, [onClose]);

  const bgColors = {
    success: 'bg-green-100 border-green-200 text-green-800',
    error: 'bg-red-100 border-red-200 text-red-800',
    warning: 'bg-yellow-100 border-yellow-200 text-yellow-800',
  };
  const Icon = type === 'success' ? CheckCircle2 : AlertCircle;

  return (
    <div className={`fixed top-4 left-1/2 transform -translate-x-1/2 z-[70] flex items-center gap-2 px-4 py-3 rounded-lg border shadow-lg animate-in fade-in slide-in-from-top-2 ${bgColors[type] || bgColors.success}`}>
      <Icon className="w-5 h-5" />
      <span className="text-sm font-medium">{message}</span>
    </div>
  );
};

// Config Modal
const ConfigModal = ({ config, onClose }) => {
  const [copied, setCopied] = useState(false);
  const codeString = `const INITIAL_DEFAULTS = ${JSON.stringify(config, null, 2)};`;

  const handleCopy = () => {
    copyToClipboard(codeString);
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  };

  return (
    <div className="fixed inset-0 z-[60] flex items-center justify-center bg-black/50 p-4 backdrop-blur-sm">
      <div className="bg-white rounded-xl shadow-2xl w-full max-w-2xl flex flex-col max-h-[90vh]">
        <div className="p-4 border-b flex justify-between items-center bg-gray-50 rounded-t-xl">
          <h3 className="font-bold text-gray-800 flex items-center gap-2"><Code className="w-5 h-5 text-indigo-600"/> 현재 설정을 기본값 코드로 변환</h3>
          <button onClick={onClose} className="p-1 hover:bg-gray-200 rounded-full"><X className="w-5 h-5"/></button>
        </div>
        <div className="p-4 overflow-hidden flex-1 flex flex-col relative">
            <pre className="absolute inset-0 p-4 overflow-auto text-xs text-green-400 bg-slate-800 font-mono custom-scrollbar m-4 rounded-lg">{codeString}</pre>
            <button onClick={handleCopy} className="absolute top-6 right-6 flex items-center gap-1.5 px-3 py-1.5 bg-white/10 hover:bg-white/20 text-white text-xs rounded backdrop-blur border border-white/20 transition">
              {copied ? <Check className="w-3 h-3"/> : <Copy className="w-3 h-3"/>} {copied ? "복사됨!" : "코드 복사"}
            </button>
        </div>
      </div>
    </div>
  );
};

// Share Modal
const ShareModal = ({ onClose, onSharePreset }) => {
  const [loading, setLoading] = useState(false);
  const [copiedType, setCopiedType] = useState(null); 
  const baseUrl = window.location.href.split('?')[0];
  
  // Check if running in a temporary preview environment
  const isPreviewEnv = window.location.hostname.includes('googleusercontent') || window.location.hostname.includes('localhost');

  const handleCopyTool = () => {
    copyToClipboard(baseUrl);
    setCopiedType('tool');
    setTimeout(() => setCopiedType(null), 2000);
  };

  const handleCopyDesign = async () => {
    setLoading(true);
    const url = await onSharePreset();
    setLoading(false);
    if (url) {
        copyToClipboard(url);
        setCopiedType('design');
        setTimeout(() => setCopiedType(null), 2000);
    }
  };

  return (
    <div className="fixed inset-0 z-[60] flex items-center justify-center bg-black/50 p-4 backdrop-blur-sm">
      <div className="bg-white rounded-xl shadow-2xl w-full max-w-md overflow-hidden animate-in fade-in zoom-in-95 duration-200">
        <div className="p-4 border-b bg-gray-50 flex justify-between items-center">
            <h3 className="font-bold text-gray-800 flex items-center gap-2"><Share2 className="w-5 h-5 text-indigo-600"/> 공유하기</h3>
            <button onClick={onClose} className="p-1 hover:bg-gray-200 rounded-full"><X className="w-5 h-5"/></button>
        </div>
        
        {/* Environment Warning */}
        {isPreviewEnv && (
            <div className="bg-yellow-50 p-3 border-b border-yellow-100 flex items-start gap-2">
                <AlertCircle className="w-4 h-4 text-yellow-600 mt-0.5 shrink-0"/>
                <div className="text-xs text-yellow-700">
                    <p className="font-bold mb-1">잠깐! 미리보기 환경입니다.</p>
                    <p>현재 주소는 비공개(Private)이므로, 공유해도 다른 사람은 볼 수 없습니다(404 오류). 공유하려면 이 코드를 <strong>외부 서버(Vercel 등)에 배포</strong>하세요.</p>
                </div>
            </div>
        )}

        <div className="p-5 space-y-4">
            <button className="w-full text-left p-4 border rounded-lg hover:border-indigo-300 hover:bg-gray-50 transition group" onClick={handleCopyTool}>
                <div className="flex justify-between items-start mb-2">
                    <div className="flex items-center gap-2 font-bold text-gray-700 group-hover:text-indigo-600"><LinkIcon className="w-4 h-4"/> 제너레이터 도구 공유</div>
                    {copiedType === 'tool' && <span className="text-xs text-green-600 font-bold bg-green-50 px-2 py-0.5 rounded-full">복사됨!</span>}
                </div>
                <div className="bg-gray-100 p-2 rounded text-[10px] text-gray-500 font-mono truncate border border-gray-200">{baseUrl}</div>
                <p className="text-[10px] text-gray-400 mt-1">설정값 없이 초기 상태의 에디터 주소를 복사합니다.</p>
            </button>
            {firebaseReady ? (
                <button className="w-full text-left p-4 border rounded-lg hover:border-indigo-400 transition group bg-indigo-50/50 border-indigo-100" onClick={handleCopyDesign} disabled={loading}>
                    <div className="flex justify-between items-start mb-2">
                        <div className="flex items-center gap-2 font-bold text-indigo-700"><Sparkles className="w-4 h-4"/> 현재 디자인 공유</div>
                        {loading ? <Loader2 className="w-4 h-4 animate-spin text-indigo-600"/> : copiedType === 'design' && <span className="text-xs text-green-600 font-bold bg-green-50 px-2 py-0.5 rounded-full">복사됨!</span>}
                    </div>
                    <div className="w-full py-2 bg-indigo-600 hover:bg-indigo-700 text-white text-xs font-bold rounded text-center transition shadow-sm">{loading ? "저장 중..." : "링크 생성 및 복사"}</div>
                    <p className="text-[10px] text-indigo-400 mt-2 text-center">현재 디자인 설정을 클라우드에 저장하고 공유 링크를 만듭니다.</p>
                </button>
            ) : (
                <div className="w-full p-4 border border-yellow-200 bg-yellow-50 rounded-lg text-xs text-yellow-800">
                    <strong>공유 기능 사용 불가</strong><br/>
                    호스팅 시 코드 상단에 Firebase 설정을 입력해야 합니다.
                </div>
            )}
        </div>
      </div>
    </div>
  );
};

// Style Panel
const StyleControlPanel = ({ fill, setFill, stroke, setStroke, strokeWidth, setStrokeWidth, useOriginal, setUseOriginal, size, setSize, isImageMode }) => {
  return (
    <div className={`space-y-4 p-3 bg-gray-50 rounded-lg border ${isImageMode ? 'opacity-40 pointer-events-none' : ''}`}>
      <div className="flex gap-4">
        <div className="flex flex-col gap-1 items-center">
          <span className="text-[10px] text-gray-500">채우기</span>
          <input type="color" value={fill} disabled={useOriginal} onChange={(e) => setFill(e.target.value)} className="w-8 h-8 border-0 p-0 rounded cursor-pointer disabled:opacity-30"/>
        </div>
        <div className="flex flex-col gap-1 items-center">
          <span className="text-[10px] text-gray-500">테두리</span>
          <input type="color" value={stroke} disabled={useOriginal} onChange={(e) => setStroke(e.target.value)} className="w-8 h-8 border-0 p-0 rounded cursor-pointer disabled:opacity-30"/>
        </div>
        <div className="flex-1 flex flex-col gap-1">
          <div className="flex justify-between text-[10px] text-gray-500"><span>테두리 두께</span><span>{strokeWidth}px</span></div>
          <input type="range" min="0" max="10" step="0.5" value={strokeWidth} disabled={useOriginal} onChange={(e) => setStrokeWidth(Number(e.target.value))} className="w-full accent-indigo-600 disabled:opacity-30" />
        </div>
      </div>
      <div className="flex justify-between items-center border-t border-gray-200 pt-2">
        <label className="text-[10px] flex items-center gap-1 cursor-pointer text-indigo-600 font-medium">
          <input type="checkbox" checked={useOriginal} onChange={(e) => setUseOriginal(e.target.checked)}/> 원본 색상/스타일 유지
        </label>
      </div>
      <div className="space-y-1 pt-2">
        <div className="flex justify-between text-xs"><span className="text-gray-600">아이콘 크기</span><span className="text-gray-400">{size}px</span></div>
        <input type="range" min="5" max="300" value={size} onChange={(e) => setSize(Number(e.target.value))} className="w-full accent-indigo-600" />
      </div>
    </div>
  );
};

const App = () => {
  // State
  const [svgContent, setSvgContent] = useState(INITIAL_DEFAULTS.svgContent); 
  const [svgUrl, setSvgUrl] = useState(null); 
  const [svgViewBox, setSvgViewBox] = useState(INITIAL_DEFAULTS.svgViewBox);
  const [svgRatio, setSvgRatio] = useState(INITIAL_DEFAULTS.svgRatio || 1);

  const [svgContent2, setSvgContent2] = useState(INITIAL_DEFAULTS.svgContent2);
  const [svgUrl2, setSvgUrl2] = useState(null);
  const [svgViewBox2, setSvgViewBox2] = useState(INITIAL_DEFAULTS.svgViewBox2);
  const [svgRatio2, setSvgRatio2] = useState(INITIAL_DEFAULTS.svgRatio2 || 1);
  const [hasSvg2, setHasSvg2] = useState(INITIAL_DEFAULTS.hasSvg2);

  const [maskImageSrc, setMaskImageSrc] = useState(null);
  const [maskImageData, setMaskImageData] = useState(null);
  
  const [patternType, setPatternType] = useState(INITIAL_DEFAULTS.patternType);
  const [renderMode, setRenderMode] = useState(INITIAL_DEFAULTS.renderMode);
  const [shapeType, setShapeType] = useState(INITIAL_DEFAULTS.shapeType);
  const [shapeText, setShapeText] = useState(INITIAL_DEFAULTS.shapeText);
  const [shapeFillMode, setShapeFillMode] = useState(INITIAL_DEFAULTS.shapeFillMode);
  const [textMaskScale, setTextMaskScale] = useState(INITIAL_DEFAULTS.textMaskScale || 0.8);
  
  const [bgColor, setBgColor] = useState(INITIAL_DEFAULTS.bgColor);
  const [fillColor, setFillColor] = useState(INITIAL_DEFAULTS.fillColor);
  const [strokeColor, setStrokeColor] = useState(INITIAL_DEFAULTS.strokeColor);
  const [strokeWidth, setStrokeWidth] = useState(INITIAL_DEFAULTS.strokeWidth);
  const [useOriginalColor, setUseOriginalColor] = useState(INITIAL_DEFAULTS.useOriginalColor);
  const [size, setSize] = useState(INITIAL_DEFAULTS.size);

  const [fillColor2, setFillColor2] = useState(INITIAL_DEFAULTS.fillColor2);
  const [strokeColor2, setStrokeColor2] = useState(INITIAL_DEFAULTS.strokeColor2);
  const [strokeWidth2, setStrokeWidth2] = useState(INITIAL_DEFAULTS.strokeWidth2);
  const [useOriginalColor2, setUseOriginalColor2] = useState(INITIAL_DEFAULTS.useOriginalColor2);
  const [size2, setSize2] = useState(INITIAL_DEFAULTS.size2);
  
  const [mixRatio, setMixRatio] = useState(INITIAL_DEFAULTS.mixRatio);
  const [patternScale, setPatternScale] = useState(INITIAL_DEFAULTS.patternScale || 1);
  const [fitContent, setFitContent] = useState(INITIAL_DEFAULTS.fitContent || false);

  const [tiltX, setTiltX] = useState(INITIAL_DEFAULTS.tiltX || 0);
  const [tiltY, setTiltY] = useState(INITIAL_DEFAULTS.tiltY || 0);
  const [zSpread, setZSpread] = useState(INITIAL_DEFAULTS.zSpread || 0);
  const [zDepthMode, setZDepthMode] = useState(INITIAL_DEFAULTS.zDepthMode || 'random');
  const [globalRotation, setGlobalRotation] = useState(INITIAL_DEFAULTS.globalRotation || 0);
  const [isAutoRotating, setIsAutoRotating] = useState(INITIAL_DEFAULTS.isAutoRotating || false);

  const [spacingX, setSpacingX] = useState(INITIAL_DEFAULTS.spacingX);
  const [spacingY, setSpacingY] = useState(INITIAL_DEFAULTS.spacingY);
  const [rotation, setRotation] = useState(INITIAL_DEFAULTS.rotation);
  const [randomRotation, setRandomRotation] = useState(INITIAL_DEFAULTS.randomRotation);
  const [randomSize, setRandomSize] = useState(INITIAL_DEFAULTS.randomSize);
  const [opacity, setOpacity] = useState(INITIAL_DEFAULTS.opacity);
  const [canvasWidth, setCanvasWidth] = useState(INITIAL_DEFAULTS.canvasWidth);
  const [canvasHeight, setCanvasHeight] = useState(INITIAL_DEFAULTS.canvasHeight);
  const [density, setDensity] = useState(INITIAL_DEFAULTS.density);

  // UI & Auth
  const [user, setUser] = useState(null);
  const [isSharing, setIsSharing] = useState(false);
  const [isPresetLoading, setIsPresetLoading] = useState(false); 
  const [isPreviewMode, setIsPreviewMode] = useState(false); // Read-only view
  const [activeTab, setActiveTab] = useState('manual');
  const [styleTab, setStyleTab] = useState('main'); 
  const [showConfigModal, setShowConfigModal] = useState(false);
  const [toast, setToast] = useState(null); 
  const [viewSubOnly, setViewSubOnly] = useState(false); 
  const [zoom, setZoom] = useState(1); 
  
  // Recording
  const [isRecording, setIsRecording] = useState(false);
  const mediaRecorderRef = useRef(null);
  const recordedChunksRef = useRef([]);
  
  const canvasRef = useRef(null);

  // --- Animation Loop ---
  useEffect(() => {
    let animationFrame;
    if (isAutoRotating) {
        const animate = () => {
            setGlobalRotation(prev => (prev + 0.5) % 360);
            animationFrame = requestAnimationFrame(animate);
        };
        animationFrame = requestAnimationFrame(animate);
    }
    return () => cancelAnimationFrame(animationFrame);
  }, [isAutoRotating]);

  // --- Recording Logic ---
  const handleStartRecording = async () => {
    try {
      const stream = await navigator.mediaDevices.getDisplayMedia({
        video: { mediaSource: "screen" },
        audio: false
      });
      
      const mediaRecorder = new MediaRecorder(stream, { mimeType: 'video/webm' });
      mediaRecorderRef.current = mediaRecorder;
      recordedChunksRef.current = [];

      mediaRecorder.ondataavailable = (event) => {
        if (event.data.size > 0) {
          recordedChunksRef.current.push(event.data);
        }
      };

      mediaRecorder.onstop = () => {
        const blob = new Blob(recordedChunksRef.current, { type: 'video/webm' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = 'pattern_recording.webm';
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        
        stream.getTracks().forEach(track => track.stop());
        setIsRecording(false);
        showToast("녹화가 완료되었습니다. (WebM)", "success");
      };

      mediaRecorder.start();
      setIsRecording(true);
      showToast("녹화 중... '공유 중지'를 누르면 저장됩니다.", "success");

      stream.getVideoTracks()[0].onended = () => {
         if (mediaRecorder.state !== 'inactive') mediaRecorder.stop();
         setIsRecording(false);
      };

    } catch (err) {
      console.error("Error: " + err);
      showToast("녹화 시작 실패 (권한 필요)", "error");
    }
  };

  const handleStopRecording = () => {
    if (mediaRecorderRef.current && isRecording) {
      mediaRecorderRef.current.stop();
    }
  };

  // --- Process SVG ---
  const processSvgContent = useCallback((svgString, idPrefix) => {
    try {
      let cleanString = svgString;
      cleanString = cleanString.replace(/<style\b[^>]*>([\s\S]*?)<\/style>/gim, "");
      cleanString = cleanString.replace(/\bid\s*=\s*["']([^"']+)["']/g, `id="${idPrefix}$1"`);
      cleanString = cleanString.replace(/url\(\s*#([^)]+)\s*\)/g, `url(#${idPrefix}$1)`);
      cleanString = cleanString.replace(/href\s*=\s*["']#([^"']+)["']/g, `href="#${idPrefix}$1"`);

      const parser = new DOMParser();
      const doc = parser.parseFromString(cleanString, "image/svg+xml");
      if (doc.querySelector("parsererror")) throw new Error("SVG 파싱 오류");

      const svgElement = doc.querySelector('svg');
      if (!svgElement) throw new Error("SVG 태그 없음");
      if (!svgElement.hasAttribute('xmlns')) svgElement.setAttribute('xmlns', "http://www.w3.org/2000/svg");

      let viewBox = '0 0 24 24';
      let ratio = 1;

      if (svgElement.hasAttribute('viewBox')) {
        viewBox = svgElement.getAttribute('viewBox');
        const parts = viewBox.split(/[\s,]+/).map(parseFloat);
        if (parts.length === 4 && parts[3] > 0) ratio = parts[2] / parts[3]; 
      } else {
         const w = parseFloat(svgElement.getAttribute('width'));
         const h = parseFloat(svgElement.getAttribute('height'));
         if (!isNaN(w) && !isNaN(h) && h > 0) { viewBox = `0 0 ${w} ${h}`; ratio = w / h; }
      }
      if (isNaN(ratio) || ratio <= 0) ratio = 1;

      return { content: svgElement.innerHTML, viewBox, serialized: new XMLSerializer().serializeToString(svgElement), ratio };
    } catch (e) {
      console.error(e);
      throw e;
    }
  }, []);

  // --- Load Preset ---
  const loadPreset = async (id) => {
      try {
          if (!db) return; 
          const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'presets', id);
          const docSnap = await getDoc(docRef);
          if (docSnap.exists()) {
              const data = docSnap.data();
              if(data.svgContent) {
                  const { content, viewBox, serialized, ratio } = processSvgContent(data.svgContent, 'loaded_m_');
                  setSvgContent(content); setSvgViewBox(viewBox); setSvgRatio(ratio);
                  setSvgUrl(URL.createObjectURL(new Blob([serialized], { type: 'image/svg+xml' })));
              }
              if(data.svgContent2) {
                  const { content, viewBox, serialized, ratio } = processSvgContent(data.svgContent2, 'loaded_s_');
                  setSvgContent2(content); setSvgViewBox2(viewBox); setSvgRatio2(ratio);
                  setSvgUrl2(URL.createObjectURL(new Blob([serialized], { type: 'image/svg+xml' })));
              }
              setHasSvg2(data.hasSvg2);
              setPatternType(data.patternType || 'grid');
              setRenderMode(data.renderMode || 'inline');
              setShapeType(data.shapeType || 'heart');
              setShapeText(data.shapeText || '♥');
              setShapeFillMode(data.shapeFillMode || 'grid');
              setTextMaskScale(data.textMaskScale || 0.8);
              setFitContent(data.fitContent || false);
              setBgColor(data.bgColor || '#f8fafc');
              setFillColor(data.fillColor); setStrokeColor(data.strokeColor); setStrokeWidth(data.strokeWidth); setUseOriginalColor(data.useOriginalColor); setSize(data.size);
              setFillColor2(data.fillColor2); setStrokeColor2(data.strokeColor2); setStrokeWidth2(data.strokeWidth2); setUseOriginalColor2(data.useOriginalColor2); setSize2(data.size2);
              setMixRatio(data.mixRatio); setPatternScale(data.patternScale);
              setSpacingX(data.spacingX); setSpacingY(data.spacingY); setRotation(data.rotation);
              setRandomRotation(data.randomRotation); setRandomSize(data.randomSize);
              setOpacity(data.opacity); setDensity(data.density);
              if(data.canvasWidth) setCanvasWidth(data.canvasWidth);
              if(data.canvasHeight) setCanvasHeight(data.canvasHeight);
              setTiltX(data.tiltX || 0); setTiltY(data.tiltY || 0); setZSpread(data.zSpread || 0); setZDepthMode(data.zDepthMode || 'random');
              setGlobalRotation(data.globalRotation || 0);
              
              setIsPreviewMode(true);
              showToast("공유된 디자인을 불러왔습니다.", "success");
          } else {
              showToast("공유된 디자인을 찾을 수 없습니다.", "error");
          }
      } catch (e) {
          console.error(e);
          showToast("설정 로드 실패", "error");
      }
  };

  // --- Auth & Init ---
  useEffect(() => {
    if (firebaseReady) {
        const initAuth = async () => {
          if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
            await signInWithCustomToken(auth, __initial_auth_token);
          } else {
            await signInAnonymously(auth);
          }
        };
        initAuth();
        const unsubscribe = onAuthStateChanged(auth, setUser);
        return () => unsubscribe();
    }
  }, []);

  useEffect(() => {
    const { content, viewBox, serialized, ratio } = processSvgContent(INITIAL_DEFAULTS.svgContent, 'def1_');
    setSvgContent(content); setSvgViewBox(viewBox); setSvgRatio(ratio);
    setSvgUrl(URL.createObjectURL(new Blob([serialized], { type: 'image/svg+xml' })));
  }, [processSvgContent]);

  useEffect(() => {
    if (user || !firebaseReady) {
        const params = new URLSearchParams(window.location.search);
        const presetId = params.get('preset');
        if (presetId && firebaseReady) {
            setIsPresetLoading(true);
            loadPreset(presetId).finally(() => setIsPresetLoading(false));
        }
    }
  }, [user]);

  useEffect(() => {
    if (shapeType === 'custom' && maskImageSrc) {
        processMaskImage(maskImageSrc);
    }
  }, [canvasWidth, canvasHeight, maskImageSrc, shapeType]);

  const showToast = (message, type = 'success') => { setToast({ message, type }); };
  const getPseudoRandom = (index, seed = 1) => { const x = Math.sin(index * 12.9898 + seed) * 43758.5453; return x - Math.floor(x); };

  const handleFileUpload = (e, isSecond = false) => {
    const file = e.target.files[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = (event) => {
      try {
        const prefix = isSecond ? `s${Math.floor(Math.random()*1000)}_` : `m${Math.floor(Math.random()*1000)}_`;
        const { content, viewBox, serialized, ratio } = processSvgContent(event.target.result, prefix);
        const blob = new Blob([serialized], { type: 'image/svg+xml' });
        const url = URL.createObjectURL(blob);
        if (isSecond) {
            setSvgContent2(serialized); setSvgViewBox2(viewBox); setSvgUrl2(url); setSvgRatio2(ratio);
            setHasSvg2(true); setStyleTab('sub'); showToast("서브 아이콘 추가됨", "success");
        } else {
            setSvgContent(serialized); setSvgViewBox(viewBox); setSvgUrl(url); setSvgRatio(ratio);
            setStyleTab('main'); showToast("메인 아이콘 변경됨", "success");
        }
      } catch (err) { showToast("파일 처리 오류", "error"); }
    };
    reader.readAsText(file);
    e.target.value = ''; 
  };

  const handleMaskUpload = (e) => {
    const file = e.target.files[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = (event) => { setMaskImageSrc(event.target.result); setShapeType('custom'); };
    reader.readAsDataURL(file);
    e.target.value = '';
  };

  const processMaskImage = (src) => {
      const img = new Image();
      img.onload = () => {
          const canvas = document.createElement('canvas');
          canvas.width = canvasWidth; canvas.height = canvasHeight;
          const ctx = canvas.getContext('2d');
          ctx.drawImage(img, 0, 0, canvasWidth, canvasHeight);
          try {
            const data = ctx.getImageData(0, 0, canvasWidth, canvasHeight).data;
            setMaskImageData(data);
          } catch(e) {}
      };
      img.src = src;
  };

  const handleExportConfig = () => {
      const currentConfig = {
          svgContent, svgViewBox, svgRatio, svgContent2, svgViewBox2, svgRatio2, hasSvg2,
          patternType, renderMode, shapeType, shapeText, shapeFillMode, textMaskScale, fitContent,
          bgColor, fillColor, strokeColor, strokeWidth, useOriginalColor, size,
          fillColor2, strokeColor2, strokeWidth2, useOriginalColor2, size2,
          mixRatio, patternScale, tiltX, tiltY, zSpread, zDepthMode, globalRotation, isAutoRotating,
          spacingX, spacingY, rotation, randomRotation, randomSize, opacity,
          canvasWidth, canvasHeight, density
      };
      setShowConfigModal(true);
  };

  const generateShareUrl = async () => {
      if (!firebaseReady || !user) { showToast("로그인이 필요합니다.", "warning"); return null; }
      try {
          const currentState = {
              svgContent, svgViewBox, svgRatio, svgContent2, svgViewBox2, svgRatio2, hasSvg2,
              patternType, renderMode, shapeType, shapeText, shapeFillMode, textMaskScale, fitContent,
              bgColor, fillColor, strokeColor, strokeWidth, useOriginalColor, size,
              fillColor2, strokeColor2, strokeWidth2, useOriginalColor2, size2,
              mixRatio, patternScale, tiltX, tiltY, zSpread, zDepthMode, globalRotation, isAutoRotating,
              spacingX, spacingY, rotation, randomRotation, randomSize, opacity,
              canvasWidth, canvasHeight, density, timestamp: new Date().toISOString()
          };
          const colRef = collection(db, 'artifacts', appId, 'public', 'data', 'presets');
          const docRef = await addDoc(colRef, currentState);
          return `${window.location.origin}${window.location.pathname}?preset=${docRef.id}`;
      } catch (e) { showToast("공유 실패", "error"); return null; }
  };

  // --- POSITION GENERATION ---
  const getTextMaskData = (width, height, text, scale) => {
    const canvas = document.createElement('canvas');
    canvas.width = width; canvas.height = height;
    const ctx = canvas.getContext('2d');
    ctx.fillStyle = 'black'; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
    ctx.font = `bold ${Math.min(width, height) * scale}px sans-serif`;
    ctx.fillText(text, width/2, height/2);
    return ctx.getImageData(0, 0, width, height).data;
  };

  const isInsideHeart = (x, y, cx, cy, s) => { const nx = (x-cx)/s; const ny = (cy-y)/s; const a = nx*nx + ny*ny - 1; return a*a*a - nx*nx*ny*ny*ny <= 0; };
  const isInsideStar = (x, y, cx, cy, rOut, rIn) => { const dx = Math.abs(x-cx); const dy = Math.abs(y-cy); if(dx+dy<=rIn) return true; if(dx>rOut||dy>rOut) return false; return dx*dx+dy*dy<=rOut*rOut; };
  const isInsideTriangle = (x, y, cx, cy, r) => { const dy = cy-y; const dx = Math.abs(cx-x); return y>=cy-r && y<=cy+r/2 && dx<=(y-(cy-r))*0.7; };
  const isInsideRhombus = (x, y, cx, cy, w, h) => { return Math.abs(x - cx)/(w/2) + Math.abs(y - cy)/(h/2) <= 1; };
  const isInsideSparkle = (x, y, cx, cy, rOut, rIn) => {
      const dx = x - cx; const dy = y - cy; const d = Math.sqrt(dx*dx + dy*dy); if (d > rOut) return false;
      const angle = Math.atan2(dy, dx) + Math.PI/2; 
      const slice = Math.PI / 2; 
      const a = (angle % slice + slice) % slice;
      const t = Math.abs((a - slice/2) / (slice/2)); 
      const threshold = rOut * (1-t) + rIn * t; 
      return d <= threshold;
  };

  const generatePositions = () => {
    const positions = [];
    const cx = canvasWidth / 2; const cy = canvasHeight / 2;
    let itemCounter = 0; 
    const focalLength = 1000;

    const addPos = (baseX, baseY) => {
        itemCounter++;
        const randRot = getPseudoRandom(itemCounter, 123);
        const randSize = getPseudoRandom(itemCounter, 456);
        const randMix = getPseudoRandom(itemCounter, 789);
        const randZ = getPseudoRandom(itemCounter, 999);

        let x = baseX - cx; let y = baseY - cy;

        // Z-Depth
        let zBase = 0;
        const nx = x / (canvasWidth/2); const ny = y / (canvasHeight/2);
        switch (zDepthMode) {
            case 'random': zBase = randZ - 0.5; break;
            case 'wave_x': zBase = Math.sin(nx * Math.PI * 2) * 0.5; break;
            case 'wave_y': zBase = Math.sin(ny * Math.PI * 2) * 0.5; break;
            case 'center_high': zBase = (1 - Math.sqrt(nx*nx + ny*ny)) * 0.5; break;
            case 'center_low': zBase = (Math.sqrt(nx*nx + ny*ny) - 1) * 0.5; break;
            case 'slope_x': zBase = nx * 0.5; break;
            case 'slope_y': zBase = ny * 0.5; break;
            default: zBase = randZ - 0.5;
        }
        let z = zBase * zSpread;

        // Global Rotation
        if (globalRotation !== 0) {
            const grRad = globalRotation * Math.PI / 180;
            const gx = x * Math.cos(grRad) - y * Math.sin(grRad);
            const gy = x * Math.sin(grRad) + y * Math.cos(grRad);
            x = gx; y = gy;
        }

        // Tilt
        const radX = (tiltX * Math.PI) / 180;
        const y1 = y * Math.cos(radX) - z * Math.sin(radX);
        const z1 = y * Math.sin(radX) + z * Math.cos(radX);
        y = y1; z = z1;
        const radY = (tiltY * Math.PI) / 180;
        const x2 = x * Math.cos(radY) + z * Math.sin(radY);
        const z2 = -x * Math.sin(radY) + z * Math.cos(radY);
        x = x2; z = z2;

        const scaleFactor = focalLength / (focalLength + z);
        const projX = x * scaleFactor + cx;
        const projY = y * scaleFactor + cy;

        positions.push({
            x: projX, y: projY, z: z, scaleFactor: scaleFactor,
            r: randomRotation ? randRot * 360 : rotation,
            s: randomSize ? 0.5 + randSize : 1, 
            isSecond: hasSvg2 && (randMix < (mixRatio / 100))
        });
    };

    let maskData = null;
    if (shapeType === 'text') maskData = getTextMaskData(canvasWidth, canvasHeight, shapeText || '?', textMaskScale);
    
    const checkPoint = (x, y) => {
        if (shapeType === 'heart') return isInsideHeart(x, y, cx, cy, Math.min(canvasWidth, canvasHeight)/3);
        if (shapeType === 'star') return isInsideStar(x, y, cx, cy, Math.min(canvasWidth, canvasHeight)*0.4, Math.min(canvasWidth, canvasHeight)*0.2);
        if (shapeType === 'triangle') return isInsideTriangle(x, y, cx, cy, Math.min(canvasWidth, canvasHeight)*0.45);
        if (shapeType === 'diamond_shape') return isInsideRhombus(x, y, cx, cy, Math.min(canvasWidth, canvasHeight)*0.8, Math.min(canvasWidth, canvasHeight)*0.8);
        if (shapeType === 'sparkle') return isInsideSparkle(x, y, cx, cy, Math.min(canvasWidth, canvasHeight)*0.45, Math.min(canvasWidth, canvasHeight)*0.15);
        if (shapeType === 'text' && maskData) { const idx = (Math.floor(y)*canvasWidth + Math.floor(x))*4+3; return maskData[idx]>100; }
        if (shapeType === 'custom' && maskImageData) { const idx = (Math.floor(y)*canvasWidth + Math.floor(x))*4+3; return maskImageData[idx]>50; }
        return false;
    };

    const effectiveScale = patternScale > 0 ? patternScale : 1;
    const baseCellSize = (hasSvg2 ? Math.max(size, size2) : size) * effectiveScale;
    const effSpaceX = spacingX * effectiveScale;
    const effSpaceY = spacingY * effectiveScale;
    
    if (patternType === 'grid') {
        const cols = Math.ceil(canvasWidth / (baseCellSize + effSpaceX)) + 1;
        const rows = Math.ceil(canvasHeight / (baseCellSize + effSpaceY)) + 1;
        for (let y = 0; y < rows; y++) {
            for (let x = 0; x < cols; x++) addPos(x * (baseCellSize + effSpaceX), y * (baseCellSize + effSpaceY));
        }
    } else if (patternType === 'brick') {
        const cols = Math.ceil(canvasWidth / (baseCellSize + effSpaceX)) + 1;
        const rows = Math.ceil(canvasHeight / (baseCellSize + effSpaceY)) + 1;
        for (let y = 0; y < rows; y++) {
            for (let x = 0; x < cols; x++) {
                const off = (y % 2 === 0) ? 0 : (baseCellSize + effSpaceX) / 2;
                addPos(x * (baseCellSize + effSpaceX) + off - baseCellSize, y * (baseCellSize + effSpaceY));
            }
        }
    } else if (patternType === 'diamond') {
        const cols = Math.ceil(canvasWidth / (baseCellSize + effSpaceX)) + 2;
        const rows = Math.ceil(canvasHeight / ((baseCellSize + effSpaceY)/2)) + 2;
        for (let y = 0; y < rows; y++) {
            for (let x = 0; x < cols; x++) {
                const off = (y % 2 === 0) ? 0 : (baseCellSize + effSpaceX) / 2;
                const py = y * (baseCellSize + effSpaceY) / 2;
                const px = x * (baseCellSize + effSpaceX) + off - baseCellSize;
                addPos(px, py);
            }
        }
    } else if (patternType === 'random') {
        for (let i = 0; i < density; i++) {
            const rx = getPseudoRandom(i, 111) * canvasWidth;
            const ry = getPseudoRandom(i, 222) * canvasHeight;
            addPos(rx, ry);
        }
    } else if (patternType === 'radial') {
        const ringCount = Math.floor(density / 10) + 1;
        for (let r = 0; r < ringCount; r++) {
            const rad = r * (baseCellSize + effSpaceX);
            const items = r === 0 ? 1 : Math.floor((2 * Math.PI * rad) / (baseCellSize + effSpaceY));
            for (let i = 0; i < items; i++) {
                const a = (i / items) * 2 * Math.PI;
                itemCounter++; 
                const randRot = getPseudoRandom(itemCounter, 123);
                const randSize = getPseudoRandom(itemCounter, 456);
                const randMix = getPseudoRandom(itemCounter, 789);
                const randZ = getPseudoRandom(itemCounter, 999);

                let x = rad * Math.cos(a); let y = rad * Math.sin(a); let z = (randZ - 0.5) * zSpread;
                if(globalRotation!==0) { const gr=globalRotation*Math.PI/180; const gx=x*Math.cos(gr)-y*Math.sin(gr); const gy=x*Math.sin(gr)+y*Math.cos(gr); x=gx; y=gy; }
                const radX = (tiltX * Math.PI)/180; const y1 = y*Math.cos(radX)-z*Math.sin(radX); const z1 = y*Math.sin(radX)+z*Math.cos(radX); y=y1; z=z1;
                const radY = (tiltY * Math.PI)/180; const x2 = x*Math.cos(radY)+z*Math.sin(radY); const z2 = -x*Math.sin(radY)+z*Math.cos(radY); x=x2; z=z2;
                const scaleFactor = 1000 / (1000 + z);
                const projX = x * scaleFactor + cx; const projY = y * scaleFactor + cy;
                const pRot = randomRotation ? randRot * 360 : (a * 180 / Math.PI) + 90 + rotation;
                
                positions.push({ x: projX, y: projY, z: z, scaleFactor: scaleFactor, r: pRot, s: randomSize ? 0.5 + randSize : 1, isSecond: hasSvg2 && (randMix < (mixRatio / 100)) });
            }
        }
    } else if (patternType === 'spiral') {
        for (let i = 0; i < density; i++) {
            const angleStep = 0.1 + (spacingY / 100); 
            const radiusStep = 0.5 + (spacingX / 10);
            const theta = i * angleStep * 5; 
            const rad = i * radiusStep;
            
            itemCounter++;
            const randRot = getPseudoRandom(itemCounter, 123);
            const randSize = getPseudoRandom(itemCounter, 456);
            const randMix = getPseudoRandom(itemCounter, 789);
            const randZ = getPseudoRandom(itemCounter, 999);
            
            let x = rad * Math.cos(theta); let y = rad * Math.sin(theta);
            
            let zBase=0; const nx=x/(canvasWidth/2); const ny=y/(canvasHeight/2);
            switch (zDepthMode) {
                case 'random': zBase = randZ - 0.5; break;
                case 'wave_x': zBase = Math.sin(nx * Math.PI * 2) * 0.5; break;
                case 'wave_y': zBase = Math.sin(ny * Math.PI * 2) * 0.5; break;
                case 'center_high': zBase = (1 - Math.sqrt(nx*nx + ny*ny)) * 0.5; break;
                case 'center_low': zBase = (Math.sqrt(nx*nx + ny*ny) - 1) * 0.5; break;
                case 'slope_x': zBase = nx * 0.5; break;
                case 'slope_y': zBase = ny * 0.5; break;
                default: zBase = randZ - 0.5;
            }
            let z = zBase * zSpread;
            if(globalRotation!==0) { const gr=globalRotation*Math.PI/180; const gx=x*Math.cos(gr)-y*Math.sin(gr); const gy=x*Math.sin(gr)+y*Math.cos(gr); x=gx; y=gy; }
            const radX = (tiltX * Math.PI)/180; const y1 = y*Math.cos(radX)-z*Math.sin(radX); const z1 = y*Math.sin(radX)+z*Math.cos(radX); y=y1; z=z1;
            const radY = (tiltY * Math.PI)/180; const x2 = x*Math.cos(radY)+z*Math.sin(radY); const z2 = -x*Math.sin(radY)+z*Math.cos(radY); x=x2; z=z2;
            const scaleFactor = 1000 / (1000 + z);
            const projX = x * scaleFactor + cx; const projY = y * scaleFactor + cy;
            const pRot = randomRotation ? randRot * 360 : (theta * 180 / Math.PI) + 90 + rotation;

            positions.push({ x: projX, y: projY, z: z, scaleFactor: scaleFactor, r: pRot, s: randomSize ? 0.5 + randSize : 1, isSecond: hasSvg2 && (randMix < (mixRatio / 100)) });
        }
    } else if (patternType === 'shape') {
        if (shapeFillMode === 'grid') {
            const cols = Math.ceil(canvasWidth / (baseCellSize + effSpaceX)) + 1;
            const rows = Math.ceil(canvasHeight / (baseCellSize + effSpaceY)) + 1;
            for (let y = 0; y < rows; y++) {
                for (let x = 0; x < cols; x++) {
                    const px = x * (baseCellSize + effSpaceX); const py = y * (baseCellSize + effSpaceY);
                    if (checkPoint(px + baseCellSize/2, py + baseCellSize/2)) addPos(px, py);
                }
            }
        } else {
            let count = 0; let attempts = 0; const maxAttempts = density * 50; 
            while (count < density && attempts < maxAttempts) {
                const rx = getPseudoRandom(attempts, 333) * canvasWidth;
                const ry = getPseudoRandom(attempts, 444) * canvasHeight;
                if (checkPoint(rx, ry)) { addPos(rx - baseCellSize/2, ry - baseCellSize/2); count++; }
                attempts++;
            }
        }
    }
    
    return positions.sort((a, b) => b.z - a.z);
  };

  const positions = generatePositions();
  
  let fitTransform = "";
  if (fitContent && positions.length > 0) {
      const margin = (hasSvg2 ? Math.max(size, size2) : size) * (patternScale || 1) * 1.5; 
      let minX = Infinity, minY = Infinity, maxX = -Infinity, maxY = -Infinity;
      positions.forEach(p => { if (p.x < minX) minX = p.x; if (p.x > maxX) maxX = p.x; if (p.y < minY) minY = p.y; if (p.y > maxY) maxY = p.y; });
      minX -= margin/2; minY -= margin/2; maxX += margin/2; maxY += margin/2;
      const contentWidth = maxX - minX; const contentHeight = maxY - minY;
      const scaleX = canvasWidth / contentWidth; const scaleY = canvasHeight / contentHeight;
      const scale = Math.min(scaleX, scaleY, 1);
      const contentCenterX = minX + contentWidth / 2; const contentCenterY = minY + contentHeight / 2;
      const canvasCenterX = canvasWidth / 2; const canvasCenterY = canvasHeight / 2;
      const tx = canvasCenterX - contentCenterX * scale; const ty = canvasCenterY - contentCenterY * scale;
      fitTransform = `translate(${tx}, ${ty}) scale(${scale})`;
  }

  const effectiveScale = patternScale > 0 ? patternScale : 1;
  const baseRenderSize = (hasSvg2 ? Math.max(size, size2) : size) * effectiveScale;

  const handleDownloadSvg = () => {
    if (!canvasRef.current) return;
    const clone = canvasRef.current.cloneNode(true);
    const serializer = new XMLSerializer();
    let source = serializer.serializeToString(clone);
    if(!source.match(/^<svg[^>]+xmlns="http\:\/\/www\.w3\.org\/2000\/svg"/)){ source = source.replace(/^<svg/, '<svg xmlns="http://www.w3.org/2000/svg"'); }
    if(!source.match(/^<svg[^>]+xmlns:xlink/)){ source = source.replace(/^<svg/, '<svg xmlns:xlink="http://www.w3.org/1999/xlink"'); }
    source = '<?xml version="1.0" standalone="no"?>\r\n' + source;
    const blob = new Blob([source], {type: "image/svg+xml;charset=utf-8"});
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.href = url; link.download = "pattern.svg";
    document.body.appendChild(link); link.click(); document.body.removeChild(link);
  };

  const handleDownload = () => {
    if (!canvasRef.current) return;
    const svgData = new XMLSerializer().serializeToString(canvasRef.current);
    const blob = new Blob([svgData], { type: "image/svg+xml;charset=utf-8" });
    const url = URL.createObjectURL(blob);
    const img = new Image();
    img.onload = () => {
      const canvas = document.createElement("canvas");
      canvas.width = canvasWidth; canvas.height = canvasHeight;
      const ctx = canvas.getContext("2d");
      ctx.fillStyle = bgColor;
      ctx.fillRect(0,0, canvasWidth, canvasHeight);
      ctx.drawImage(img, 0, 0);
      const link = document.createElement("a");
      link.href = canvas.toDataURL("image/png");
      link.download = "pattern.png";
      document.body.appendChild(link); link.click(); document.body.removeChild(link);
    };
    img.src = url;
  };

  if (isPresetLoading) {
      return (
          <div className="flex h-screen items-center justify-center bg-gray-50 flex-col gap-3">
              <Loader2 className="w-10 h-10 animate-spin text-indigo-600"/>
              <p className="text-sm font-medium text-gray-600">설정을 불러오는 중...</p>
          </div>
      );
  }

  // --- PREVIEW MODE RENDER ---
  if (isPreviewMode) {
      return (
        <div className="flex flex-col h-screen bg-gray-50 text-gray-800 font-sans overflow-hidden">
             {toast && <Toast message={toast.message} type={toast.type} onClose={() => setToast(null)} />}
             <header className="bg-white border-b px-6 py-4 flex items-center justify-between z-10 shadow-sm">
                <div className="flex items-center gap-2 text-indigo-600">
                    <Layers className="w-6 h-6" />
                    <h1 className="text-xl font-bold tracking-tight">SVG 패턴 생성기 Pro <span className="text-xs bg-indigo-100 px-2 py-0.5 rounded-full ml-1">Viewer</span></h1>
                </div>
                <div className="flex gap-2">
                    <button onClick={() => setIsPreviewMode(false)} className="flex items-center gap-2 px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white text-sm font-medium rounded-lg shadow-sm transition">
                        <Edit3 className="w-4 h-4"/> <span>편집하기 (Remix)</span>
                    </button>
                </div>
            </header>
            <main className="flex-1 bg-gray-100 flex items-center justify-center p-8 overflow-hidden relative">
                <div className="relative shadow-2xl transition-transform duration-500 ease-out origin-center" style={{ transform: `scale(${Math.min(1, (window.innerWidth - 64) / canvasWidth)})` }}>
                     {/* Reuse SVG Render Logic (Ideally separate component but inline for now) */}
                    <svg ref={canvasRef} width={canvasWidth} height={canvasHeight} style={{ backgroundColor: bgColor }} className="block bg-white" xmlns="http://www.w3.org/2000/svg" xmlnsXlink="http://www.w3.org/1999/xlink">
                        <defs>
                            <style>{`
                                ${!useOriginalColor ? `.pattern-main path, .pattern-main rect, .pattern-main circle, .pattern-main ellipse, .pattern-main line, .pattern-main polyline, .pattern-main polygon { fill: ${fillColor} !important; stroke: ${strokeColor} !important; stroke-width: ${strokeWidth}px !important; }` : ''}
                                ${!useOriginalColor2 ? `.pattern-sub path, .pattern-sub rect, .pattern-sub circle, .pattern-sub ellipse, .pattern-sub line, .pattern-sub polyline, .pattern-sub polygon { fill: ${fillColor2} !important; stroke: ${strokeColor2} !important; stroke-width: ${strokeWidth2}px !important; }` : ''}
                            `}</style>
                        </defs>
                        <g transform={fitTransform}>
                            {positions.map((pos, index) => {
                                const isSub = pos.isSecond;
                                const currentUrl = isSub ? svgUrl2 : svgUrl;
                                const currentRaw = isSub ? svgContent2 : svgContent;
                                const conf = isSub ? { fill: fillColor2, stroke: strokeColor2, sw: strokeWidth2, orig: useOriginalColor2, baseSz: size2, ratio: svgRatio2 } : { fill: fillColor, stroke: strokeColor, sw: strokeWidth, orig: useOriginalColor, baseSz: size, ratio: svgRatio };
                                const innerHtml = currentRaw ? currentRaw.replace(/<svg[^>]*>|<\/svg>/g, "") : "";
                                const currentViewBox = isSub ? svgViewBox2 : svgViewBox;
                                const effectiveScale = patternScale > 0 ? patternScale : 1;
                                const scale3D = pos.scaleFactor || 1;
                                const finalScale = effectiveScale * scale3D;
                                const scaledBaseSz = conf.baseSz * finalScale;
                                let w, h;
                                if (conf.ratio > 1) { w = scaledBaseSz * pos.s; h = (scaledBaseSz / conf.ratio) * pos.s; } else { h = scaledBaseSz * pos.s; w = (scaledBaseSz * conf.ratio) * pos.s; }
                                const cx = w/2; const cy = h/2;
                                const tx = pos.x - w/2; const ty = pos.y - h/2;
                                return (
                                <g key={index} transform={`translate(${tx}, ${ty}) rotate(${pos.r}, ${cx}, ${cy})`} style={{ opacity: opacity }} className={isSub ? "pattern-sub" : "pattern-main"}>
                                {renderMode === 'image' && currentUrl ? ( <image href={currentUrl} width={w} height={h} /> ) : ( <svg width={w} height={h} viewBox={currentViewBox} dangerouslySetInnerHTML={{ __html: innerHtml }} style={{ overflow: 'visible', }} /> )}
                                </g>
                            )})}
                        </g>
                    </svg>
                </div>
            </main>
        </div>
      );
  }

  // --- EDITOR MODE RENDER ---
  return (
    <div className="flex flex-col h-screen bg-gray-50 text-gray-800 font-sans overflow-hidden">
      {toast && <Toast message={toast.message} type={toast.type} onClose={() => setToast(null)} />}
      {isSharing && <ShareModal onClose={() => setIsSharing(false)} onSharePreset={generateShareUrl} />}
      {showConfigModal && <ConfigModal config={{
          svgContent, svgViewBox, svgRatio, svgContent2, svgViewBox2, svgRatio2, hasSvg2,
          patternType, renderMode, shapeType, shapeText, shapeFillMode, textMaskScale, fitContent,
          bgColor, fillColor, strokeColor, strokeWidth, useOriginalColor, size,
          fillColor2, strokeColor2, strokeWidth2, useOriginalColor2, size2,
          mixRatio, patternScale, tiltX, tiltY, zSpread, zDepthMode, globalRotation, isAutoRotating,
          spacingX, spacingY, rotation, randomRotation, randomSize, opacity,
          canvasWidth, canvasHeight, density
      }} onClose={() => setShowConfigModal(false)} />}

      <header className="bg-white border-b px-6 py-4 flex items-center justify-between z-10 shadow-sm">
        <div className="flex items-center gap-2 text-indigo-600">
          <Layers className="w-6 h-6" />
          <h1 className="text-xl font-bold tracking-tight">SVG 패턴 생성기 Pro</h1>
        </div>
        <div className="flex gap-2">
          <button 
            onClick={() => setIsSharing(true)} 
            className="flex items-center gap-2 px-3 py-2 bg-indigo-50 hover:bg-indigo-100 text-indigo-700 text-sm font-medium rounded-lg transition"
          >
            <Share2 className="w-4 h-4" />
            <span>공유하기</span>
          </button>
          
          <button onClick={() => !isRecording ? handleStartRecording() : handleStopRecording()} className={`flex items-center gap-2 px-3 py-2 border rounded-lg text-sm font-medium transition ${isRecording ? 'bg-red-50 border-red-200 text-red-600 animate-pulse' : 'bg-white hover:bg-gray-50 text-gray-700'}`}>
            {isRecording ? <Square className="w-4 h-4 fill-current"/> : <Video className="w-4 h-4"/>}
            <span>{isRecording ? "중지" : "동영상"}</span>
          </button>

          <div className="w-px h-8 bg-gray-200 mx-1"></div>
          <button onClick={handleDownload} className="flex items-center gap-2 px-3 py-2 bg-gray-100 hover:bg-gray-200 text-gray-700 text-sm font-medium rounded-lg transition"><FileImage className="w-4 h-4" /><span>PNG</span></button>
          <button onClick={handleDownloadSvg} className="flex items-center gap-2 px-3 py-2 bg-indigo-600 hover:bg-indigo-700 text-white text-sm font-medium rounded-lg shadow-sm transition"><FileCode className="w-4 h-4" /><span>SVG</span></button>
        </div>
      </header>

      <div className="flex flex-1 overflow-hidden">
        {/* SIDEBAR */}
        <aside className="w-80 bg-white border-r flex flex-col">
          <div className="flex border-b">
             <button onClick={() => setActiveTab('manual')} className={`flex-1 py-3 text-sm font-medium flex items-center justify-center gap-2 ${activeTab === 'manual' ? 'text-indigo-600 border-b-2 border-indigo-600 bg-indigo-50/50' : 'text-gray-500 hover:text-gray-700'}`}>
                <Layers className="w-4 h-4"/> 에디터
             </button>
             <button onClick={() => setActiveTab('ai')} className={`flex-1 py-3 text-sm font-medium flex items-center justify-center gap-2 ${activeTab === 'ai' ? 'text-purple-600 border-b-2 border-purple-600 bg-purple-50/50' : 'text-gray-500 hover:text-gray-700'}`}>
                <Sparkles className="w-4 h-4"/> AI 스튜디오
             </button>
          </div>

          <div className="flex-1 overflow-y-auto custom-scrollbar p-6 space-y-8">
            {activeTab === 'ai' ? (
                <div className="space-y-8 animate-in fade-in slide-in-from-left-4 duration-300">
                    <div className="space-y-3">
                        <div className="flex items-center gap-2 mb-1">
                             <Sparkles className="w-5 h-5 text-purple-600"/>
                             <label className="text-sm font-bold text-gray-700 uppercase tracking-wide">AI 패턴 매직</label>
                        </div>
                        <p className="text-xs text-gray-500">
                            원하는 분위기나 스타일을 설명하면 AI가 자동으로 패턴 설정을 생성해줍니다.
                        </p>
                        <textarea 
                            value={aiPrompt}
                            onChange={(e) => setAiPrompt(e.target.value)}
                            placeholder="예: 파스텔톤의 부드러운 봄 느낌, 사이버펑크 네온 스타일, 복잡한 기하학적 무늬..."
                            className="w-full h-24 p-3 text-sm border rounded-lg focus:ring-2 focus:ring-purple-500 outline-none resize-none bg-purple-50/30"
                        />
                        <button 
                            onClick={generatePatternWithAI}
                            disabled={isGeneratingPattern || !aiPrompt.trim()}
                            className="w-full py-2.5 bg-gradient-to-r from-purple-600 to-indigo-600 hover:from-purple-700 hover:to-indigo-700 text-white text-sm font-medium rounded-lg shadow-md transition flex items-center justify-center gap-2 disabled:opacity-50"
                        >
                            {isGeneratingPattern ? <Loader2 className="w-4 h-4 animate-spin"/> : <Wand2 className="w-4 h-4"/>}
                            {isGeneratingPattern ? "생성 중..." : "AI 패턴 생성하기 ✨"}
                        </button>
                    </div>
                    <div className="border-t border-dashed border-gray-200 my-4"></div>
                    <div className="space-y-3">
                        <div className="flex items-center gap-2 mb-1">
                             <Grid className="w-5 h-5 text-indigo-600"/>
                             <label className="text-sm font-bold text-gray-700 uppercase tracking-wide">AI 아이콘 생성</label>
                        </div>
                        <p className="text-xs text-gray-500">
                            패턴에 사용할 아이콘이 없나요? AI에게 그려달라고 해보세요.
                        </p>
                         <input 
                            type="text"
                            value={aiIconPrompt}
                            onChange={(e) => setAiIconPrompt(e.target.value)}
                            onKeyDown={(e) => e.key === 'Enter' && generateIconWithAI()}
                            placeholder="예: 귀여운 고양이 얼굴, 심플한 번개..."
                            className="w-full p-3 text-sm border rounded-lg focus:ring-2 focus:ring-indigo-500 outline-none bg-indigo-50/30"
                        />
                        <button 
                            onClick={generateIconWithAI}
                            disabled={isGeneratingIcon || !aiIconPrompt.trim()}
                            className="w-full py-2.5 bg-white border border-indigo-200 text-indigo-700 hover:bg-indigo-50 text-sm font-medium rounded-lg transition flex items-center justify-center gap-2 disabled:opacity-50"
                        >
                            {isGeneratingIcon ? <Loader2 className="w-4 h-4 animate-spin"/> : <Sparkles className="w-4 h-4"/>}
                            {isGeneratingIcon ? "생성 중..." : "AI 아이콘 만들기 ✨"}
                        </button>
                    </div>
                </div>
            ) : (
                <div className="space-y-6 animate-in fade-in slide-in-from-right-4">
                    <div className="space-y-2 p-3 bg-indigo-50 border border-indigo-100 rounded-lg">
                        <div className="flex justify-between text-xs">
                            <span className="font-bold text-indigo-800 flex items-center gap-1"><Maximize className="w-3 h-3"/> 패턴 배율</span>
                            <span className="text-indigo-600 font-mono">{patternScale}x</span>
                        </div>
                        <input type="range" min="0.1" max="3" step="0.1" value={patternScale} onChange={(e) => setPatternScale(Number(e.target.value))} className="w-full accent-indigo-600" />
                    </div>

                    <div className="space-y-3">
                        <label className="text-xs font-bold text-gray-500 uppercase tracking-wider">아이콘 모듈</label>
                        <div className="flex gap-2">
                            <div className="flex-1 relative group">
                                <label className={`flex flex-col items-center justify-center w-full h-20 border-2 border-dashed rounded-lg cursor-pointer transition overflow-hidden ${styleTab === 'main' ? 'border-indigo-500 bg-indigo-50' : 'border-gray-300 hover:border-gray-400'}`}>
                                    {svgUrl ? <img src={svgUrl} className="w-8 h-8 opacity-80 object-contain" alt="icon1"/> : <Upload className="w-6 h-6 text-gray-400"/>}
                                    <span className="text-[10px] text-gray-500 mt-1">메인 아이콘</span>
                                    <input type="file" accept=".svg" onChange={(e) => handleFileUpload(e, false)} className="hidden" />
                                </label>
                            </div>
                            <div className="flex-1 relative group">
                                <label className={`flex flex-col items-center justify-center w-full h-20 border-2 border-dashed rounded-lg cursor-pointer transition overflow-hidden ${styleTab === 'sub' && hasSvg2 ? 'border-indigo-500 bg-indigo-50' : 'border-gray-300 hover:border-gray-400'}`}>
                                    {hasSvg2 ? (
                                        <>
                                            <img src={svgUrl2} className="w-8 h-8 opacity-80 object-contain" alt="icon2"/>
                                            <button onClick={(e)=>{e.preventDefault(); setHasSvg2(false); setSvgContent2(null); setStyleTab('main');}} className="absolute top-1 right-1 p-1 bg-white rounded-full shadow hover:bg-red-50 text-red-500"><X className="w-3 h-3"/></button>
                                        </>
                                    ) : ( <Plus className="w-6 h-6 text-gray-400"/> )}
                                    <span className="text-[10px] text-gray-500 mt-1">서브 아이콘 (믹스)</span>
                                    <input type="file" accept=".svg" onChange={(e) => handleFileUpload(e, true)} className="hidden" />
                                </label>
                            </div>
                        </div>
                    </div>

                    <div className="space-y-3">
                        <div className="flex justify-between items-center"><label className="text-xs font-bold text-gray-500 uppercase tracking-wider">스타일 & 크기</label></div>
                        <div className="flex bg-gray-100 p-1 rounded-lg mb-2">
                            <button onClick={() => {setStyleTab('main'); setViewSubOnly(false);}} className={`flex-1 py-1.5 text-xs font-medium rounded-md transition ${styleTab === 'main' ? 'bg-white shadow text-indigo-600' : 'text-gray-500'}`}>메인 아이콘</button>
                            <button onClick={() => hasSvg2 ? setStyleTab('sub') : showToast("서브 아이콘을 먼저 업로드하세요.", "warning")} className={`flex-1 py-1.5 text-xs font-medium rounded-md transition ${styleTab === 'sub' ? 'bg-white shadow text-pink-600' : 'text-gray-400'} ${!hasSvg2 ? 'opacity-50 cursor-not-allowed' : ''}`}>서브 아이콘</button>
                        </div>
                        {hasSvg2 && styleTab === 'sub' && (
                            <div className="mb-2 flex items-center justify-end">
                                <label className="text-[10px] flex items-center gap-1 cursor-pointer bg-pink-50 px-2 py-1 rounded text-pink-600 font-bold border border-pink-100">
                                    <input type="checkbox" checked={viewSubOnly} onChange={(e) => setViewSubOnly(e.target.checked)}/> <Monitor className="w-3 h-3"/> 서브 아이콘만 보기
                                </label>
                            </div>
                        )}
                        {hasSvg2 && !viewSubOnly && (
                            <div className="px-1 pb-4 border-b border-dashed border-gray-200">
                                <div className="flex justify-between text-xs mb-1">
                                    <span className="flex items-center gap-1 text-gray-600"><Percent className="w-3 h-3"/> 믹스 비율</span>
                                    <span className="text-gray-400">{mixRatio}%</span>
                                </div>
                                <div className="flex items-center gap-2">
                                    <span className="text-[10px] text-indigo-400">Main</span>
                                    <input type="range" min="0" max="100" value={mixRatio} onChange={(e) => setMixRatio(Number(e.target.value))} className="w-full accent-indigo-600" />
                                    <span className="text-[10px] text-pink-400">Sub</span>
                                </div>
                            </div>
                        )}
                        {styleTab === 'main' ? (
                        <StyleControlPanel fill={fillColor} setFill={setFillColor} stroke={strokeColor} setStroke={setStrokeColor} strokeWidth={strokeWidth} setStrokeWidth={setStrokeWidth} useOriginal={useOriginalColor} setUseOriginal={setUseOriginalColor} size={size} setSize={setSize} isImageMode={renderMode === 'image'}/>
                        ) : (
                        <StyleControlPanel fill={fillColor2} setFill={setFillColor2} stroke={strokeColor2} setStroke={setStrokeColor2} strokeWidth={strokeWidth2} setStrokeWidth={setStrokeWidth2} useOriginal={useOriginalColor2} setUseOriginal={setUseOriginalColor2} size={size2} setSize={setSize2} isImageMode={renderMode === 'image'}/>
                        )}
                        <label className="flex items-center gap-2 text-xs text-gray-600 cursor-pointer pt-1 px-1">
                            <input type="checkbox" checked={randomSize} onChange={(e) => setRandomSize(e.target.checked)} className="rounded text-indigo-600"/>
                            <span>무작위 크기 변형</span>
                        </label>
                    </div>

                    <div className="space-y-3">
                        <label className="text-xs font-bold text-gray-500 uppercase tracking-wider">패턴 유형</label>
                        <div className="grid grid-cols-6 gap-1">
                            {['grid', 'brick', 'diamond', 'radial', 'spiral', 'shape'].map(t => (
                                <button key={t} onClick={() => setPatternType(t)} className={`flex flex-col items-center justify-center p-2 rounded-lg border transition ${patternType === t ? 'border-indigo-500 bg-indigo-50 text-indigo-700' : 'border-gray-200 hover:border-gray-300'}`}>
                                    {t === 'grid' && <LayoutGrid className="w-4 h-4 mb-1"/>}
                                    {t === 'brick' && <Grid className="w-4 h-4 mb-1"/>}
                                    {t === 'diamond' && <Diamond className="w-4 h-4 mb-1"/>}
                                    {t === 'radial' && <RefreshCcw className="w-4 h-4 mb-1"/>}
                                    {t === 'random' && <Shuffle className="w-4 h-4 mb-1"/>}
                                    {t === 'spiral' && <Wind className="w-4 h-4 mb-1"/>}
                                    {t === 'shape' && <Heart className="w-4 h-4 mb-1"/>}
                                    <span className="text-[10px] capitalize">{t}</span>
                                </button>
                            ))}
                        </div>
                    </div>

                    {patternType === 'shape' && (
                    <div className="space-y-3 p-3 bg-gray-50 rounded-lg border border-gray-100">
                        <label className="text-xs font-bold text-gray-500 uppercase tracking-wider">모양 마스크</label>
                        <div className="grid grid-cols-5 gap-1">
                            {['heart', 'star', 'triangle', 'diamond_shape', 'sparkle', 'text', 'custom'].map(t => (
                                <button key={t} onClick={() => setShapeType(t)} className={`p-2 rounded border flex justify-center ${shapeType === t ? 'bg-indigo-100 border-indigo-400 text-indigo-700' : 'bg-white'}`}>
                                    {t === 'heart' && <Heart className="w-4 h-4"/>}
                                    {t === 'star' && <Star className="w-4 h-4"/>}
                                    {t === 'triangle' && <Triangle className="w-4 h-4"/>}
                                    {t === 'diamond_shape' && <Diamond className="w-4 h-4"/>}
                                    {t === 'sparkle' && <Sparkles className="w-4 h-4"/>}
                                    {t === 'text' && <Type className="w-4 h-4"/>}
                                    {t === 'custom' && <ImageIcon className="w-4 h-4"/>}
                                </button>
                            ))}
                        </div>
                        {shapeType === 'text' && (
                            <div className="space-y-2">
                                <input type="text" value={shapeText} onChange={(e) => setShapeText(e.target.value)} className="w-full px-2 py-1 border rounded text-sm outline-none" placeholder="Text..."/>
                                <div className="flex items-center gap-2"><span className="text-xs text-gray-500 w-16">텍스트 크기</span><input type="range" min="0.2" max="2.0" step="0.1" value={textMaskScale} onChange={(e) => setTextMaskScale(Number(e.target.value))} className="flex-1 accent-indigo-600"/></div>
                            </div>
                        )}
                        {shapeType === 'custom' && (
                            <div className="flex flex-col gap-2">
                                <label className="flex items-center gap-2 w-full p-2 border border-dashed bg-white rounded cursor-pointer text-xs text-gray-500 hover:text-indigo-600 hover:border-indigo-400 transition">
                                    <Upload className="w-3 h-3"/><span>{maskImageSrc ? "이미지 변경" : "이미지 업로드 (실루엣)"}</span>
                                    <input type="file" accept="image/*" onChange={handleMaskUpload} className="hidden" />
                                </label>
                                {maskImageSrc && <img src={maskImageSrc} className="h-10 object-contain opacity-50 border rounded bg-white" alt="mask preview"/>}
                            </div>
                        )}
                        <div className="flex gap-2 text-xs pt-2 border-t border-gray-200">
                                <button onClick={() => setShapeFillMode('grid')} className={`flex-1 py-1 rounded ${shapeFillMode === 'grid' ? 'bg-indigo-600 text-white' : 'bg-white border'}`}>규칙적</button>
                                <button onClick={() => setShapeFillMode('random')} className={`flex-1 py-1 rounded ${shapeFillMode === 'random' ? 'bg-indigo-600 text-white' : 'bg-white border'}`}>랜덤</button>
                        </div>
                    </div>
                    )}

                    <div className="space-y-3 p-3 bg-purple-50 rounded-lg border border-purple-100">
                        <label className="text-xs font-bold text-purple-700 uppercase tracking-wider flex items-center gap-1"><Box className="w-3 h-3"/> 3D 효과 & 변형</label>
                        <div className="space-y-2">
                            <div className="flex justify-between text-[10px] text-purple-600"><span>입체 회전 (Tilt)</span></div>
                            <div className="flex gap-2">
                                <div className="flex-1"><label className="text-[9px] text-gray-400 block mb-0.5 text-center">X축</label><input type="range" min="-60" max="60" value={tiltX} onChange={(e) => setTiltX(Number(e.target.value))} className="w-full accent-purple-500" /></div>
                                <div className="flex-1"><label className="text-[9px] text-gray-400 block mb-0.5 text-center">Y축</label><input type="range" min="-60" max="60" value={tiltY} onChange={(e) => setTiltY(Number(e.target.value))} className="w-full accent-purple-500" /></div>
                            </div>
                            <div className="pt-1">
                                <div className="flex justify-between text-[10px] text-purple-600 mb-1"><span>Z축 깊이</span><span>{zSpread}</span></div>
                                <input type="range" min="0" max="1000" step="50" value={zSpread} onChange={(e) => setZSpread(Number(e.target.value))} className="w-full accent-purple-500" />
                                <div className="grid grid-cols-4 gap-1 mt-2">
                                    <button onClick={() => setZDepthMode('random')} className={`p-1 rounded text-[9px] border ${zDepthMode==='random'?'bg-purple-100 border-purple-300 text-purple-700':'bg-white text-gray-500'}`}>랜덤</button>
                                    <button onClick={() => setZDepthMode('wave_x')} className={`p-1 rounded text-[9px] border ${zDepthMode==='wave_x'?'bg-purple-100 border-purple-300 text-purple-700':'bg-white text-gray-500'}`}>물결X</button>
                                    <button onClick={() => setZDepthMode('center_high')} className={`p-1 rounded text-[9px] border ${zDepthMode==='center_high'?'bg-purple-100 border-purple-300 text-purple-700':'bg-white text-gray-500'}`}>산</button>
                                    <button onClick={() => setZDepthMode('slope_x')} className={`p-1 rounded text-[9px] border ${zDepthMode==='slope_x'?'bg-purple-100 border-purple-300 text-purple-700':'bg-white text-gray-500'}`}>경사</button>
                                </div>
                            </div>
                            <div className="pt-2 border-t border-purple-200">
                                <div className="flex justify-between items-center mb-1">
                                    <span className="text-[10px] text-purple-600">Z축 회전 (Spin)</span>
                                    <label className="text-[9px] flex items-center gap-1 cursor-pointer bg-purple-100 px-2 py-0.5 rounded text-purple-700 font-bold">
                                        <input type="checkbox" checked={isAutoRotating} onChange={(e) => setIsAutoRotating(e.target.checked)} className="rounded text-purple-600 w-3 h-3"/>
                                        {isAutoRotating ? <StopCircle className="w-3 h-3"/> : <PlayCircle className="w-3 h-3"/>} 자동 회전
                                    </label>
                                </div>
                                <input type="range" min="0" max="360" value={globalRotation} onChange={(e) => setGlobalRotation(Number(e.target.value))} className="w-full accent-purple-500" />
                            </div>
                        </div>
                    </div>

                    <div className="space-y-4">
                        <div className="flex justify-between items-center"><span className="text-xs font-bold text-gray-500">배경색</span><input type="color" value={bgColor} onChange={(e) => setBgColor(e.target.value)} className="w-6 h-6 border-0 p-0 rounded cursor-pointer"/></div>
                        {(patternType === 'grid' || patternType === 'brick' || patternType === 'diamond' || (patternType === 'shape' && shapeFillMode === 'grid')) && (
                            <>
                                <div className="space-y-1"><div className="flex justify-between text-xs"><span className="text-gray-600">가로 간격</span><span className="text-gray-400">{spacingX}px</span></div><input type="range" min="0" max="100" value={spacingX} onChange={(e) => setSpacingX(Number(e.target.value))} className="w-full accent-indigo-600" /></div>
                                <div className="space-y-1"><div className="flex justify-between text-xs"><span className="text-gray-600">세로 간격</span><span className="text-gray-400">{spacingY}px</span></div><input type="range" min="0" max="100" value={spacingY} onChange={(e) => setSpacingY(Number(e.target.value))} className="w-full accent-indigo-600" /></div>
                            </>
                        )}
                        {patternType === 'spiral' && (
                            <>
                                <div className="space-y-1"><div className="flex justify-between text-xs"><span className="text-gray-600">퍼짐 정도 (Spread)</span><span className="text-gray-400">{spacingX}</span></div><input type="range" min="1" max="100" value={spacingX} onChange={(e) => setSpacingX(Number(e.target.value))} className="w-full accent-indigo-600" /></div>
                                <div className="space-y-1"><div className="flex justify-between text-xs"><span className="text-gray-600">회전 각도 (Angle Step)</span><span className="text-gray-400">{spacingY}</span></div><input type="range" min="1" max="100" value={spacingY} onChange={(e) => setSpacingY(Number(e.target.value))} className="w-full accent-indigo-600" /></div>
                            </>
                        )}
                        {(patternType === 'random' || patternType === 'radial' || patternType === 'spiral' || (patternType === 'shape' && shapeFillMode === 'random')) && (
                                <div className="space-y-1"><div className="flex justify-between text-xs"><span className="text-gray-600">밀도 (개수)</span><span className="text-gray-400">{density}</span></div><input type="range" min="10" max="1000" value={density} onChange={(e) => setDensity(Number(e.target.value))} className="w-full accent-indigo-600" /></div>
                        )}
                        <div className="space-y-1"><div className="flex justify-between text-xs"><span className="text-gray-600">개별 아이콘 회전</span><span className="text-gray-400">{rotation}°</span></div><input type="range" min="0" max="360" value={rotation} onChange={(e) => setRotation(Number(e.target.value))} className="w-full accent-indigo-600" /><label className="flex items-center gap-2 text-xs text-gray-600 cursor-pointer pt-1"><input type="checkbox" checked={randomRotation} onChange={(e) => setRandomRotation(e.target.checked)} className="rounded text-indigo-600"/><span>무작위 방향</span></label></div>
                    </div>

                    <div className="space-y-4 pt-4 border-t border-gray-200">
                        <div className="flex justify-between items-center"><span className="text-xs font-bold text-gray-500 uppercase tracking-wider">캔버스 크기</span></div>
                        <div className="flex gap-2">
                            <div className="flex-1"><label className="text-[10px] text-gray-500 block mb-1">너비</label><input type="number" value={canvasWidth} onChange={(e) => setCanvasWidth(Number(e.target.value))} className="w-full px-2 py-1.5 border rounded text-xs" /></div>
                            <div className="flex-1"><label className="text-[10px] text-gray-500 block mb-1">높이</label><input type="number" value={canvasHeight} onChange={(e) => setCanvasHeight(Number(e.target.value))} className="w-full px-2 py-1.5 border rounded text-xs" /></div>
                        </div>
                        <div className="flex justify-between items-center mt-2"><label className="text-[10px] flex items-center gap-1 cursor-pointer text-indigo-600 font-bold"><input type="checkbox" checked={fitContent} onChange={(e) => setFitContent(e.target.checked)} className="rounded text-indigo-600"/><span>캔버스에 맞춤 (Fit to Canvas)</span></label></div>
                        <div className="grid grid-cols-3 gap-1">
                            <button onClick={() => { setCanvasWidth(1080); setCanvasHeight(1080); }} className="px-2 py-1 border rounded text-[10px] hover:bg-gray-50 flex flex-col items-center"><span className="font-bold">1:1</span><span className="text-[9px] text-gray-400">Insta</span></button>
                            <button onClick={() => { setCanvasWidth(1920); setCanvasHeight(1080); }} className="px-2 py-1 border rounded text-[10px] hover:bg-gray-50 flex flex-col items-center"><span className="font-bold">16:9</span><span className="text-[9px] text-gray-400">FHD</span></button>
                            <button onClick={() => { setCanvasWidth(2480); setCanvasHeight(3508); }} className="px-2 py-1 border rounded text-[10px] hover:bg-gray-50 flex flex-col items-center"><span className="font-bold">A4</span><span className="text-[9px] text-gray-400">Print</span></button>
                        </div>
                    </div>
                </div>
            )}
            
            <div className="pt-4 border-t mt-4 space-y-2">
                <button onClick={handleExportConfig} className="w-full py-2 flex items-center justify-center gap-2 text-sm text-indigo-600 bg-indigo-50 hover:bg-indigo-100 rounded-lg transition"><Code className="w-4 h-4" /><span>{'< >'} 현재 설정을 기본값 코드로 변환</span></button>
                <button onClick={() => {
                        setSvgContent(INITIAL_DEFAULTS.svgContent); setSvgRatio(INITIAL_DEFAULTS.svgRatio || 1);
                        setSvgContent2(INITIAL_DEFAULTS.svgContent2); setSvgRatio2(INITIAL_DEFAULTS.svgRatio2 || 1); setHasSvg2(INITIAL_DEFAULTS.hasSvg2);
                        setPatternType(INITIAL_DEFAULTS.patternType);
                        setSize(INITIAL_DEFAULTS.size); setSize2(INITIAL_DEFAULTS.size2);
                        setFillColor(INITIAL_DEFAULTS.fillColor); setStrokeColor(INITIAL_DEFAULTS.strokeColor); setStrokeWidth(INITIAL_DEFAULTS.strokeWidth); setUseOriginalColor(INITIAL_DEFAULTS.useOriginalColor);
                        setFillColor2(INITIAL_DEFAULTS.fillColor2); setStrokeColor2(INITIAL_DEFAULTS.strokeColor2); setStrokeWidth2(INITIAL_DEFAULTS.strokeWidth2); setUseOriginalColor2(INITIAL_DEFAULTS.useOriginalColor2);
                        setMixRatio(INITIAL_DEFAULTS.mixRatio); setCanvasWidth(INITIAL_DEFAULTS.canvasWidth); setCanvasHeight(INITIAL_DEFAULTS.canvasHeight);
                        setPatternScale(1); setFitContent(INITIAL_DEFAULTS.fitContent); setTextMaskScale(INITIAL_DEFAULTS.textMaskScale);
                        setTiltX(0); setTiltY(0); setZSpread(0); setGlobalRotation(0); setIsAutoRotating(false); setZDepthMode('random');
                        showToast("설정 초기화됨", "success");
                    }} className="w-full py-2 flex items-center justify-center gap-2 text-sm text-red-500 hover:bg-red-50 rounded-lg transition"><Trash2 className="w-4 h-4" /><span>초기화</span></button>
            </div>
          </div>
        </aside>

        {/* MAIN CANVAS */}
        <main className="flex-1 bg-gray-200 overflow-hidden relative">
          <div className="absolute inset-0 flex items-center justify-center overflow-auto p-8 custom-scrollbar">
            <div className="relative shadow-2xl transition-transform duration-200 ease-out origin-center" style={{ transform: `scale(${zoom})`, }}>
                <svg ref={canvasRef} width={canvasWidth} height={canvasHeight} style={{ backgroundColor: bgColor }} className="block bg-white transition-colors duration-200" xmlns="http://www.w3.org/2000/svg" xmlnsXlink="http://www.w3.org/1999/xlink">
                <defs>
                    <style>
                    {`
                        ${!useOriginalColor ? `.pattern-main path, .pattern-main rect, .pattern-main circle, .pattern-main ellipse, .pattern-main line, .pattern-main polyline, .pattern-main polygon { fill: ${fillColor} !important; stroke: ${strokeColor} !important; stroke-width: ${strokeWidth}px !important; }` : ''}
                        ${!useOriginalColor2 ? `.pattern-sub path, .pattern-sub rect, .pattern-sub circle, .pattern-sub ellipse, .pattern-sub line, .pattern-sub polyline, .pattern-sub polygon { fill: ${fillColor2} !important; stroke: ${strokeColor2} !important; stroke-width: ${strokeWidth2}px !important; }` : ''}
                    `}
                    </style>
                </defs>
                <g transform={fitTransform}>
                    {positions.map((pos, index) => {
                        if (viewSubOnly && !pos.isSecond) return null;
                        const isSub = pos.isSecond;
                        const currentUrl = isSub ? svgUrl2 : svgUrl;
                        const currentRaw = isSub ? svgContent2 : svgContent;
                        const conf = isSub ? { fill: fillColor2, stroke: strokeColor2, sw: strokeWidth2, orig: useOriginalColor2, baseSz: size2, ratio: svgRatio2 } : { fill: fillColor, stroke: strokeColor, sw: strokeWidth, orig: useOriginalColor, baseSz: size, ratio: svgRatio };
                        const innerHtml = currentRaw ? currentRaw.replace(/<svg[^>]*>|<\/svg>/g, "") : "";
                        const currentViewBox = isSub ? svgViewBox2 : svgViewBox;
                        const effectiveScale = patternScale > 0 ? patternScale : 1;
                        const scale3D = pos.scaleFactor || 1;
                        const finalScale = effectiveScale * scale3D;
                        const scaledBaseSz = conf.baseSz * finalScale;
                        let w, h;
                        if (conf.ratio > 1) { w = scaledBaseSz * pos.s; h = (scaledBaseSz / conf.ratio) * pos.s; } else { h = scaledBaseSz * pos.s; w = (scaledBaseSz * conf.ratio) * pos.s; }
                        const cx = w/2; const cy = h/2;
                        const tx = pos.x - w/2; const ty = pos.y - h/2;
                        return (
                        <g key={index} transform={`translate(${tx}, ${ty}) rotate(${pos.r}, ${cx}, ${cy})`} style={{ opacity: opacity }} className={isSub ? "pattern-sub" : "pattern-main"}>
                        {renderMode === 'image' && currentUrl ? ( <image href={currentUrl} width={w} height={h} /> ) : ( <svg width={w} height={h} viewBox={currentViewBox} dangerouslySetInnerHTML={{ __html: innerHtml }} style={{ overflow: 'visible', }} /> )}
                        </g>
                    )})}
                </g>
                </svg>
                <div className="absolute -bottom-8 right-0 text-xs text-gray-500 bg-white/80 px-2 py-1 rounded backdrop-blur-sm pointer-events-none">{canvasWidth} x {canvasHeight} px</div>
            </div>
          </div>
          <div className="absolute top-4 right-4 z-20 flex gap-2">
                <div className="bg-white/90 backdrop-blur p-1 rounded-lg shadow border flex text-xs">
                    <button onClick={() => setRenderMode('inline')} className={`px-2 py-1 rounded ${renderMode === 'inline' ? 'bg-indigo-100 text-indigo-700 font-bold' : 'text-gray-500'}`}>색상모드</button>
                    <button onClick={() => setRenderMode('image')} className={`px-2 py-1 rounded ${renderMode === 'image' ? 'bg-indigo-100 text-indigo-700 font-bold' : 'text-gray-500'}`}>이미지모드</button>
                </div>
          </div>
          <div className="absolute bottom-4 left-4 z-20">
                <div className="bg-white/90 backdrop-blur p-1.5 rounded-lg shadow border flex items-center gap-2">
                    <button onClick={() => setZoom(z => Math.max(0.1, z - 0.1))} className="p-1 hover:bg-gray-100 rounded text-gray-600"><ZoomOut className="w-4 h-4"/></button>
                    <span className="text-xs w-10 text-center font-medium text-gray-600">{Math.round(zoom * 100)}%</span>
                    <button onClick={() => setZoom(z => Math.min(3, z + 0.1))} className="p-1 hover:bg-gray-100 rounded text-gray-600"><ZoomIn className="w-4 h-4"/></button>
                    <button onClick={() => setZoom(1)} className="text-[9px] text-gray-400 hover:text-gray-600 px-1 border-l ml-1">100%</button>
                </div>
          </div>
        </main>
      </div>
    </div>
  );
};

export default App;
