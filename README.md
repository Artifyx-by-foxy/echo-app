<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Echo - The Kindness Chain</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif; }
        .safe-area-bottom { padding-bottom: constant(safe-area-inset-bottom); padding-bottom: env(safe-area-inset-bottom); }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        .animate-fade-in { animation: fadeIn 0.4s ease-out; }
        @keyframes bounceIn {
          0% { transform: scale(0.9) translateY(10px); opacity: 0; }
          60% { transform: scale(1.05) translateY(-5px); opacity: 1; }
          100% { transform: scale(1) translateY(0); }
        }
        .animate-bounce-in { animation: bounceIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
    </style>
</head>
<body>
    <div id="root"></div>

    <!-- React and Babel CDNs -->
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

    <!-- React Application -->
    <script type="text/babel">
        // Replace empty string with your Gemini API key from Google AI Studio if running on your own domain
        const apiKey = ""; 

        const MOCK_USER = {
          id: 'u1',
          name: 'Alex Rivera',
          avatar: 'https://images.unsplash.com/photo-1534528741775-53994a69daeb?w=150&auto=format&fit=crop&q=80',
          karma: 850,
          echoes_started: 12,
          impact_score: 45,
          location: 'Brooklyn, NY'
        };

        const INITIAL_REQUESTS = [
          {
            id: 1,
            user: { name: 'Sarah Chen', avatar: 'https://images.unsplash.com/photo-1494790108377-be9c29b29330?w=150&auto=format&fit=crop&q=80' },
            title: 'Borrow a power drill',
            description: 'Just need to hang two shelves. Will return within 2 hours!',
            distance: '0.2 mi',
            time: '20m ago',
            tags: ['Tools', 'Quick'],
            urgency: 'medium',
            status: 'open'
          },
          {
            id: 2,
            user: { name: 'Marcus Johnson', avatar: 'https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?w=150&auto=format&fit=crop&q=80' },
            title: 'Help moving a couch',
            description: 'I have a van, just need an extra pair of hands to lift it down one flight of stairs.',
            distance: '0.5 mi',
            time: '1h ago',
            tags: ['Labor', 'Moving'],
            urgency: 'high',
            status: 'open'
          },
          {
            id: 3,
            user: { name: 'Emma Wilson', avatar: 'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?w=150&auto=format&fit=crop&q=80' },
            title: 'Cat sitting for weekend',
            description: 'Going away for 2 days. Just need someone to feed Luna twice.',
            distance: '0.8 mi',
            time: '3h ago',
            tags: ['Pets', 'Weekend'],
            urgency: 'low',
            status: 'open'
          },
        ];

        const IMPACT_CHAINS = [
          {
            id: 'c1',
            starter: 'You',
            action: 'Walked Dog',
            date: 'Oct 24',
            nodes: [
              { id: 1, name: 'You', action: 'Walked Dog', status: 'root' },
              { id: 2, name: 'Sarah', action: 'Baked Cookies', status: 'child' },
              { id: 3, name: 'Mike', action: 'Fixed Bike', status: 'grandchild' },
              { id: 4, name: 'Jenny', action: 'Tutored Math', status: 'grandchild' },
              { id: 5, name: 'Tom', action: 'Moved Box', status: 'great-grandchild' },
            ]
          }
        ];

        // --- SVG Icons ---
        const Heart = ({ size = 24, className = '', fill = 'none' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill={fill} stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M19 14c1.49-1.46 3-3.21 3-5.5A5.5 5.5 0 0 0 16.5 3c-1.76 0-3 .5-4.5 2-1.5-1.5-2.74-2-4.5-2A5.5 5.5 0 0 0 2 8.5c0 2.3 1.5 4.05 3 5.5l7 7Z"/></svg>
        );
        const MapPin = ({ size = 24, className = '' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z"/><circle cx="12" cy="10" r="3"/></svg>
        );
        const MessageCircle = ({ size = 24, className = '' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="m3 21 1.9-5.7a8.5 8.5 0 1 1 3.8 3.8z"/></svg>
        );
        const Share2 = ({ size = 24, className = '' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><circle cx="18" cy="5" r="3"/><circle cx="6" cy="12" r="3"/><circle cx="18" cy="19" r="3"/><line x1="8.59" x2="15.42" y1="13.51" y2="17.49"/><line x1="15.41" x2="8.59" y1="6.51" y2="10.49"/></svg>
        );
        const User = ({ size = 24, className = '' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M19 21v-2a4 4 0 0 0-4-4H9a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
        );
        const Plus = ({ size = 24, className = '' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M5 12h14"/><path d="M12 5v14"/></svg>
        );
        const Search = ({ size = 24, className = '' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><circle cx="11" cy="11" r="8"/><path d="m21 21-4.3-4.3"/></svg>
        );
        const Bell = ({ size = 24, className = '' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M6 8a6 6 0 0 1 12 0c0 7 3 9 3 9H3s3-2 3-9"/><path d="M10.3 21a1.9 1.9 0 0 0 3.4 0"/></svg>
        );
        const Award = ({ size = 24, className = '' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="m15.4 17.7-.8-.7a5.5 5.5 0 0 0-7.3 0l-.8.7c-2.8 2.3-4.5 1.5-4.5-3.2v-3.7A5.5 5.5 0 0 1 12 4.5 5.5 5.5 0 0 1 20 7v3.7c0 4.7-1.7 5.5-4.6 3.2Z"/><path d="M12 4.5V2"/></svg>
        );
        const Filter = ({ size = 24, className = '' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polygon points="22 3 2 3 10 12.46 10 19 14 21 14 12.46 22 3"/></svg>
        );
        const CheckCircle = ({ size = 24, className = '' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="m9 11 3 3L22 4"/></svg>
        );
        const Activity = ({ size = 24, className = '' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M22 12h-4l-3 9L9 3l-3 9H2"/></svg>
        );
        const Sparkles = ({ size = 24, className = '' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="m12 3-1.912 5.813a2 2 0 0 1-1.275 1.275L3 12l5.813 1.912a2 2 0 0 1 1.275 1.275L12 21l1.912-5.813a2 2 0 0 1 1.275-1.275L21 12l-5.813-1.912a2 2 0 0 1-1.275-1.275L12 3Z"/></svg>
        );
        const X = ({ size = 24, className = '' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M18 6 6 18"/><path d="m6 6 12 12"/></svg>
        );
        const ArrowRight = ({ size = 24, className = '' }) => (
          <svg className={className} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M5 12h14"/><path d="m12 5 7 7-7 7"/></svg>
        );
        const Loader2 = ({ size = 24, className = '' }) => (
          <svg className={`animate-spin ${className}`} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M21 12a9 9 0 1 1-6.219-8.56"/></svg>
        );

        // --- Gemini API Call with Retry Logic ---
        const callGemini = async (prompt) => {
          if (!apiKey) {
            console.warn("No Gemini API key supplied. Skipping AI call.");
            return null;
          }
          const maxRetries = 5;
          let delay = 1000;
          for (let i = 0; i < maxRetries; i++) {
            try {
              const res = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ contents: [{ parts: [{ text: prompt }] }] })
              });
              if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
              const data = await res.json();
              return data.candidates?.[0]?.content?.parts?.[0]?.text || null;
            } catch (err) {
              if (i === maxRetries - 1) return null;
              await new Promise(r => setTimeout(r, delay));
              delay *= 2;
            }
          }
          return null;
        };

        // --- Components ---
        const Badge = ({ children, type = 'neutral' }) => {
          const colors = {
            neutral: 'bg-gray-100 text-gray-600',
            primary: 'bg-indigo-100 text-indigo-600',
            urgent: 'bg-red-100 text-red-600',
            success: 'bg-green-100 text-green-700',
          };
          return <span className={`px-2 py-0.5 rounded-full text-xs font-medium ${colors[type]}`}>{children}</span>;
        };

        const NavItem = ({ icon: Icon, label, active, onClick }) => (
          <button onClick={onClick} className={`flex flex-col items-center justify-center w-full py-2 transition-colors ${active ? 'text-indigo-600' : 'text-gray-400 hover:text-gray-600'}`}>
            <Icon size={24} />
            <span className="text-[10px] mt-1 font-medium">{label}</span>
          </button>
        );

        const RequestCard = ({ req, onAccept }) => (
          <div className="bg-white p-4 rounded-xl shadow-sm border border-gray-100 mb-4 hover:shadow-md transition-shadow">
            <div className="flex justify-between items-start mb-2">
              <div className="flex items-center space-x-3">
                <img src={req.user.avatar} alt={req.user.name} className="w-10 h-10 rounded-full object-cover border border-gray-200" onError={(e) => { e.target.src = "https://images.unsplash.com/photo-1535713875002-d1d0cf377fde?w=150"; }} />
                <div>
                  <h3 className="font-semibold text-gray-900 text-sm">{req.user.name}</h3>
                  <div className="flex items-center text-xs text-gray-500">
                    <MapPin size={12} className="mr-1" /> {req.distance} • {req.time}
                  </div>
                </div>
              </div>
              {req.urgency === 'high' && <Badge type="urgent">Urgent</Badge>}
            </div>
            <h4 className="font-bold text-gray-800 mb-1">{req.title}</h4>
            <p className="text-gray-600 text-sm mb-3">{req.description}</p>
            <div className="flex flex-wrap gap-2 mb-4">
              {req.tags.map(tag => (
                <span key={tag} className="text-xs text-gray-500 bg-gray-50 px-2 py-1 rounded-md border border-gray-100">#{tag}</span>
              ))}
            </div>
            <div className="flex items-center justify-between pt-3 border-t border-gray-50">
              <button className="text-gray-400 hover:text-gray-600 transition-colors"><MessageCircle size={20} /></button>
              <button onClick={() => onAccept(req.id)} className="bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-2 rounded-full text-sm font-semibold shadow-lg shadow-indigo-200 transition-all active:scale-95 flex items-center">
                <Heart size={16} className="mr-2" fill="white" /> Help Out
              </button>
            </div>
          </div>
        );

        const EchoChainVisualizer = ({ chain }) => {
          const [story, setStory] = React.useState("Your initial act of kindness allowed Sarah to bake cookies, which she shared with Mike, giving him the energy to fix a bike!");
          const [isGenerating, setIsGenerating] = React.useState(false);

          const generateStory = async () => {
            setIsGenerating(true);
            const chainDescription = chain.nodes.map(n => `${n.name} ${n.action}`).join(' -> ');
            const res = await callGemini(`Write a short, whimsical 2-sentence story about how these kindness actions connect: ${chainDescription}. Start with "It all began when..."`);
            if (res) setStory(res.trim());
            setIsGenerating(false);
          };

          return (
            <div className="bg-gradient-to-br from-indigo-50 to-purple-50 p-6 rounded-2xl border border-indigo-100 mb-6 relative overflow-hidden">
              <div className="absolute top-0 right-0 p-4 opacity-10"><Activity size={120} /></div>
              <div className="relative z-10">
                <div className="flex justify-between items-end mb-6">
                  <div>
                    <h3 className="text-lg font-bold text-gray-900">Your "{chain.action}" Echo</h3>
                    <p className="text-indigo-600 text-sm font-medium">Started {chain.date} • Impacted 5 people</p>
                  </div>
                  <button className="bg-white text-indigo-600 px-3 py-1.5 rounded-lg text-xs font-bold border border-indigo-100 shadow-sm flex items-center hover:bg-indigo-50">
                    <Share2 size={14} className="mr-1" /> Share
                  </button>
                </div>
                <div className="flex items-center justify-between relative mb-6">
                  <div className="absolute top-1/2 left-0 w-full h-1 bg-indigo-200 -z-0 rounded-full" />
                  {chain.nodes.slice(0, 4).map((node, idx) => (
                    <div key={node.id} className="relative z-10 flex flex-col items-center group">
                      <div className={`w-10 h-10 rounded-full flex items-center justify-center border-2 shadow-sm ${node.status === 'root' ? 'bg-indigo-600 border-indigo-600 text-white' : 'bg-white border-indigo-200 text-indigo-600'}`}>
                        {node.status === 'root' ? <User size={20} /> : <Heart size={16} fill="currentColor" />}
                      </div>
                      <div className="absolute -bottom-8 w-24 text-center opacity-0 group-hover:opacity-100 transition-opacity bg-gray-900 text-white text-[10px] py-1 px-2 rounded pointer-events-none">
                        {node.name} • {node.action}
                      </div>
                    </div>
                  ))}
                  <div className="relative z-10 flex flex-col items-center">
                     <div className="w-8 h-8 rounded-full bg-indigo-100 border-2 border-indigo-300 flex items-center justify-center text-indigo-600 text-xs font-bold">+1</div>
                  </div>
                </div>
                <div className="mt-4 bg-white/80 backdrop-blur-sm p-4 rounded-xl border border-white/50 shadow-sm">
                  <div className="flex justify-between items-start mb-2">
                    <h4 className="text-xs font-bold text-indigo-800 flex items-center uppercase tracking-wide"><Sparkles size={12} className="mr-1" /> Impact Story</h4>
                    <button onClick={generateStory} disabled={isGenerating} className="text-[10px] bg-indigo-100 hover:bg-indigo-200 text-indigo-700 px-2 py-1 rounded-md font-medium disabled:opacity-50 flex items-center">
                      {isGenerating ? <Loader2 size={12} className="mr-1" /> : <Sparkles size={12} className="mr-1" />}
                      {isGenerating ? 'Dreaming...' : 'Refresh Story'}
                    </button>
                  </div>
                  <p className="text-sm text-gray-700 italic leading-relaxed">"{story}"</p>
                </div>
              </div>
            </div>
          );
        };

        const CreateRequestModal = ({ onClose, onPost }) => {
          const [draft, setDraft] = React.useState('');
          const [title, setTitle] = React.useState('');
          const [isPolishing, setIsPolishing] = React.useState(false);

          const handlePolish = async () => {
            if (!draft) return;
            setIsPolishing(true);
            const res = await callGemini(`Rewrite this request into a warm, polite favor ask for a neighborhood app (under 200 chars): "${draft}". Don't use quotes or hashtags.`);
            if (res) setDraft(res.trim());
            setIsPolishing(false);
          };

          return (
            <div className="fixed inset-0 z-50 flex items-center justify-center bg-black/50 backdrop-blur-sm animate-fade-in p-4">
              <div className="bg-white w-full max-w-sm rounded-3xl shadow-2xl overflow-hidden transform animate-bounce-in">
                <div className="bg-gray-50 px-6 py-4 border-b border-gray-100 flex justify-between items-center">
                  <h3 className="font-bold text-gray-900">Ask for a Favor</h3>
                  <button onClick={onClose} className="text-gray-400 hover:text-gray-600 p-1"><X size={20} /></button>
                </div>
                <div className="p-6 space-y-4">
                  <div>
                    <label className="block text-xs font-bold text-gray-500 uppercase tracking-wide mb-1">Title</label>
                    <input value={title} onChange={(e) => setTitle(e.target.value)} placeholder="e.g., Need a ladder" className="w-full bg-gray-50 border border-gray-200 rounded-xl px-4 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500" />
                  </div>
                  <div>
                    <label className="block text-xs font-bold text-gray-500 uppercase tracking-wide mb-1">Details</label>
                    <textarea value={draft} onChange={(e) => setDraft(e.target.value)} placeholder="Describe what you need help with..." className="w-full h-32 bg-gray-50 border border-gray-200 rounded-xl px-4 py-3 text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 resize-none" />
                  </div>
                # echo-app