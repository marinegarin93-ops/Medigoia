# Medigoia
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MEDIGO - Assistant Médical</title>
    <meta name="description" content="Votre assistant médical personnel - Questions santé, conseils et médecins à proximité">
    
    <!-- PWA Meta Tags -->
    <link rel="manifest" href="#" id="manifest-placeholder">
    <meta name="theme-color" content="#3B82F6">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="default">
    <meta name="apple-mobile-web-app-title" content="MEDIGO">
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest/dist/umd/lucide.js"></script>
    
    <!-- React & ReactDOM -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

    <style>
        /* Splash Screen */
        .splash-screen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 9999;
            transition: opacity 0.5s ease-out;
        }
        
        .splash-logo {
            animation: pulse 2s infinite;
        }
        
        @keyframes pulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.1); }
        }
        
        .fade-out {
            opacity: 0;
            pointer-events: none;
        }
        
        /* Custom scrollbar */
        ::-webkit-scrollbar {
            width: 6px;
        }
        
        ::-webkit-scrollbar-track {
            background: #f1f1f1;
        }
        
        ::-webkit-scrollbar-thumb {
            background: #3B82F6;
            border-radius: 3px;
        }
    </style>
</head>
<body>
    <!-- Splash Screen -->
    <div id="splash" class="splash-screen">
        <div class="text-center">
            <div class="splash-logo w-32 h-32 bg-white rounded-full flex items-center justify-center mb-4 shadow-2xl">
                <span class="text-6xl">🏥</span>
            </div>
            <h1 class="text-4xl font-bold text-white mb-2">MEDIGO</h1>
            <p class="text-white/80">Assistant Médical</p>
            <div class="mt-6 flex justify-center space-x-1">
                <div class="w-2 h-2 bg-white rounded-full animate-bounce"></div>
                <div class="w-2 h-2 bg-white rounded-full animate-bounce" style="animation-delay: 0.1s;"></div>
                <div class="w-2 h-2 bg-white rounded-full animate-bounce" style="animation-delay: 0.2s;"></div>
            </div>
        </div>
    </div>

    <!-- App Container -->
    <div id="app"></div>

    <!-- PWA Installation Prompt -->
    <div id="install-prompt" style="display: none;" class="fixed bottom-4 left-4 right-4 bg-gradient-to-r from-blue-500 to-purple-600 text-white p-4 rounded-2xl shadow-xl z-50">
        <div class="flex items-center space-x-3">
            <span class="text-2xl">📱</span>
            <div class="flex-1">
                <h3 class="font-bold text-sm">Installer MEDIGO</h3>
                <p class="text-xs text-blue-100">Ajoutez l'app à votre écran d'accueil !</p>
            </div>
            <button id="install-btn" class="bg-white text-blue-600 px-4 py-2 rounded-xl font-bold text-sm hover:bg-blue-50 transition-colors">
                Installer
            </button>
            <button id="dismiss-btn" class="text-blue-100 hover:text-white">✕</button>
        </div>
    </div>

    <script type="text/babel">
        const { useState, useRef, useEffect } = React;
        const { MessageCircle, Send, Phone, Calendar, User, Home, MapPin, Navigation, Star, Stethoscope, Clock, Heart, AlertTriangle, CheckCircle } = lucide;

        const MedigoApp = () => {
            const [currentView, setCurrentView] = useState('home');
            const [messages, setMessages] = useState([
                {
                    id: 1,
                    text: "Salut ! 👋 Je suis MEDIGO, ton assistant médical sympa ! Comment puis-je t'aider aujourd'hui ?",
                    sender: 'bot',
                    timestamp: new Date(Date.now() - 5000)
                }
            ]);
            const [inputText, setInputText] = useState('');
            const [isTyping, setIsTyping] = useState(false);
            const messagesEndRef = useRef(null);

            const scrollToBottom = () => {
                messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
            };

            useEffect(() => {
                scrollToBottom();
            }, [messages]);

            // Médecins fictifs avec géolocalisation
            const nearbyDoctors = [
                {
                    id: 1,
                    name: "Dr. Marie Dubois",
                    specialty: "Médecin Généraliste",
                    rating: 4.8,
                    distance: "350m",
                    address: "15 rue de la Santé, 75014 Paris",
                    phone: "01 45 67 89 12",
                    nextAvailable: "Aujourd'hui 14h30",
                    avatar: "👩‍⚕️"
                },
                {
                    id: 2,
                    name: "Dr. Pierre Martin",
                    specialty: "Médecin Généraliste",
                    rating: 4.6,
                    distance: "680m",
                    address: "8 avenue des Lilas, 75014 Paris",
                    phone: "01 42 33 55 77",
                    nextAvailable: "Demain 9h15",
                    avatar: "👨‍⚕️"
                },
                {
                    id: 3,
                    name: "Dr. Sophie Bernard",
                    specialty: "Pédiatre",
                    rating: 4.9,
                    distance: "1.2km",
                    address: "22 boulevard Saint-Michel, 75006 Paris",
                    phone: "01 46 78 90 34",
                    nextAvailable: "Demain 16h00",
                    avatar: "👩‍⚕️"
                }
            ];

            // Base de connaissances médicale avec emojis
            const medicalKnowledge = {
                'mal de tête': {
                    response: "Oh là ! Mal de tête ? 🤕 Pas de panique ! Voici ce que tu peux essayer : \n• 💧 Boire beaucoup d'eau (déshydratation courante !) \n• 😴 Te reposer dans un endroit calme \n• ❄️ Appliquer une compresse froide \n• 💊 Prendre du paracétamol si besoin",
                    severity: 'low',
                    recommendation: "Si ça persiste plus de 2 jours ou empire, file voir un doc ! 👨‍⚕️"
                },
                'fièvre': {
                    response: "Aïe aïe ! De la fièvre ? 🌡️ Allez, on gère ça ensemble : \n• 💦 Hydrate-toi à fond ! \n• 😴 Repos au max \n• 👕 Vêtements légers \n• 📊 Surveille ta température",
                    severity: 'medium',
                    recommendation: "Si ça dépasse 38.5°C ou dure plus de 3 jours, direction le médecin ! 🏃‍♂️"
                },
                'toux': {
                    response: "Cette toux t'embête ? 😷 J'ai des tips qui marchent : \n• 🍯 Miel + boissons chaudes (magique !) \n• 💨 Humidificateur d'air \n• 🚭 Évite la fumée \n• 🍬 Pastilles pour la gorge",
                    severity: 'low',
                    recommendation: "Si ça traîne plus d'une semaine ou avec du sang, consulte vite ! 🩺"
                },
                'rhume': {
                    response: "Rhume en vue ? 🤧 On va te remettre d'aplomb : \n• 😴 Repos et hydratation \n• 💨 Inhalations vapeur \n• 🌊 Lavage nasal sérum physio \n• 🍋 Miel-citron chaud (un délice !)",
                    severity: 'low',
                    recommendation: "Normalement ça passe en 7-10 jours, sinon va voir ton doc ! 👩‍⚕️"
                }
            };

            const quickActions = [
                { icon: MessageCircle, text: 'Chat Médical', color: 'from-blue-400 to-blue-600', emoji: '💬' },
                { icon: MapPin, text: 'Médecins proches', color: 'from-green-400 to-green-600', emoji: '📍' },
                { icon: Heart, text: 'Conseils Santé', color: 'from-pink-400 to-pink-600', emoji: '💖' },
                { icon: Phone, text: 'Urgences', color: 'from-red-400 to-red-600', emoji: '🚨' }
            ];

            const healthTips = [
                { icon: '💧', title: 'Hydratation', tip: 'Bois 1.5L d\'eau par jour minimum !' },
                { icon: '🏃‍♂️', title: 'Activité', tip: '30 min de marche quotidienne = top forme !' },
                { icon: '😴', title: 'Sommeil', tip: '7-8h de sommeil pour être au top !' },
                { icon: '🥗', title: 'Nutrition', tip: '5 fruits et légumes par jour, c\'est parti !' }
            ];

            const analyzeMessage = (message) => {
                const lowerMessage = message.toLowerCase();
                
                for (const [symptom, data] of Object.entries(medicalKnowledge)) {
                    if (lowerMessage.includes(symptom)) {
                        return {
                            found: true,
                            response: data.response,
                            severity: data.severity,
                            recommendation: data.recommendation
                        };
                    }
                }

                const emergencyKeywords = ['douleur poitrine', 'difficulté respirer', 'évanouissement', 'sang', 'accident'];
                const hasEmergency = emergencyKeywords.some(keyword => lowerMessage.includes(keyword));

                if (hasEmergency) {
                    return {
                        found: true,
                        response: "🚨 ALERTE ! Ces symptômes peuvent être très sérieux !",
                        severity: 'high',
                        recommendation: "Appelle immédiatement le 15 ou fonce aux urgences ! Ne perds pas de temps ! 🏥"
                    };
                }

                return {
                    found: false,
                    response: "Hmm, je comprends ta préoccupation ! 🤔 Pour des conseils sur-mesure, le mieux c'est de voir un pro qui connaît ton dossier ! 👨‍⚕️✨",
                    severity: 'medium',
                    recommendation: "Je te conseille de prendre RDV avec ton médecin traitant ! 📅"
                };
            };

            const sendMessage = () => {
                if (!inputText.trim()) return;

                const userMessage = {
                    id: messages.length + 1,
                    text: inputText,
                    sender: 'user',
                    timestamp: new Date()
                };

                setMessages(prev => [...prev, userMessage]);
                setInputText('');
                setIsTyping(true);

                setTimeout(() => {
                    const analysis = analyzeMessage(inputText);
                    
                    const botResponse = {
                        id: messages.length + 2,
                        text: analysis.response,
                        sender: 'bot',
                        timestamp: new Date(),
                        severity: analysis.severity,
                        recommendation: analysis.recommendation
                    };

                    setMessages(prev => [...prev, botResponse]);
                    setIsTyping(false);
                }, 1500);
            };

            const formatTime = (date) => {
                return date.toLocaleTimeString('fr-FR', { 
                    hour: '2-digit', 
                    minute: '2-digit' 
                });
            };

            const getSeverityColor = (severity) => {
                switch (severity) {
                    case 'low': return 'text-green-600 bg-green-100';
                    case 'medium': return 'text-amber-600 bg-amber-100';
                    case 'high': return 'text-red-600 bg-red-100';
                    default: return 'text-gray-600 bg-gray-100';
                }
            };

            const getSeverityIcon = (severity) => {
                switch (severity) {
                    case 'low': return '✅';
                    case 'medium': return '⚠️';
                    case 'high': return '🚨';
                    default: return '💡';
                }
            };

            const HomeView = () => (
                <div className="p-6 space-y-6 bg-gradient-to-br from-blue-50 to-purple-50 min-h-full">
                    <div className="text-center">
                        <div className="w-24 h-24 bg-gradient-to-br from-blue-500 via-purple-500 to-pink-500 rounded-3xl mx-auto mb-4 flex items-center justify-center transform rotate-3 hover:rotate-0 transition-transform duration-300 shadow-xl">
                            <span className="text-4xl">🏥</span>
                        </div>
                        <h1 className="text-3xl font-bold bg-gradient-to-r from-blue-600 to-purple-600 bg-clip-text text-transparent mb-2">
                            MEDIGO
                        </h1>
                        <p className="text-gray-600 font-medium">Ton assistant santé préféré ! ✨</p>
                    </div>

                    <div className="bg-gradient-to-r from-blue-500 to-purple-600 rounded-2xl p-6 text-white shadow-xl">
                        <div className="flex items-center mb-3">
                            <span className="text-2xl mr-3">👋</span>
                            <h3 className="font-bold text-lg">Salut champion !</h3>
                        </div>
                        <p className="text-blue-100">
                            Prêt à prendre soin de ta santé ? Je suis là pour t'aider avec tes questions, 
                            te donner des conseils et te trouver les meilleurs médecins près de chez toi ! 💪
                        </p>
                    </div>

                    <div className="grid grid-cols-2 gap-4">
                        {quickActions.map((action, index) => (
                            <button
                                key={index}
                                onClick={() => {
                                    if (action.text === 'Chat Médical') setCurrentView('chat');
                                    if (action.text === 'Médecins proches') setCurrentView('doctors');
                                }}
                                className="group relative p-6 rounded-2xl bg-white shadow-lg hover:shadow-2xl transition-all duration-300 transform hover:-translate-y-1"
                            >
                                <div className={`w-16 h-16 bg-gradient-to-br ${action.color} rounded-2xl mx-auto mb-4 flex items-center justify-center shadow-lg group-hover:scale-110 transition-transform`}>
                                    <span className="text-2xl">{action.emoji}</span>
                                </div>
                                <span className="text-sm font-semibold text-gray-700 block">{action.text}</span>
                            </button>
                        ))}
                    </div>

                    <div className="bg-white rounded-2xl p-6 shadow-lg">
                        <div className="flex items-center mb-4">
                            <span className="text-2xl mr-3">💡</span>
                            <h3 className="font-bold text-gray-800">Conseils du jour</h3>
                        </div>
                        <div className="grid grid-cols-2 gap-3">
                            {healthTips.map((tip, index) => (
                                <div key={index} className="bg-gradient-to-br from-gray-50 to-gray-100 rounded-xl p-3">
                                    <div className="text-xl mb-1">{tip.icon}</div>
                                    <div className="text-xs font-semibold text-gray-700 mb-1">{tip.title}</div>
                                    <div className="text-xs text-gray-600">{tip.tip}</div>
                                </div>
                            ))}
                        </div>
                    </div>
                </div>
            );

            const ChatView = () => (
                <div className="flex flex-col h-full bg-gradient-to-br from-blue-50 to-purple-50">
                    <div className="bg-gradient-to-r from-blue-500 via-purple-500 to-pink-500 text-white p-6 rounded-b-3xl shadow-xl">
                        <div className="flex items-center space-x-3">
                            <div className="w-12 h-12 bg-white/20 rounded-2xl flex items-center justify-center">
                                <span className="text-2xl">🤖</span>
                            </div>
                            <div>
                                <h2 className="text-xl font-bold">Dr. MEDIGO</h2>
                                <p className="text-blue-100 text-sm">Ton assistant médical sympa ! ✨</p>
                            </div>
                        </div>
                    </div>

                    <div className="flex-1 overflow-y-auto p-4 space-y-4">
                        {messages.map((message) => (
                            <div
                                key={message.id}
                                className={`flex ${message.sender === 'user' ? 'justify-end' : 'justify-start'}`}
                            >
                                <div
                                    className={`max-w-xs lg:max-w-md px-5 py-4 rounded-3xl shadow-lg ${
                                        message.sender === 'user'
                                            ? 'bg-gradient-to-r from-blue-500 to-purple-600 text-white'
                                            : 'bg-white text-gray-800 border border-gray-100'
                                    }`}
                                >
                                    <p className="text-sm whitespace-pre-line">{message.text}</p>
                                    
                                    {message.recommendation && (
                                        <div className={`mt-4 p-4 rounded-2xl ${getSeverityColor(message.severity)} border-l-4 border-current`}>
                                            <div className="flex items-center space-x-2 mb-2">
                                                <span className="text-lg">{getSeverityIcon(message.severity)}</span>
                                                <span className="text-xs font-bold">Conseil MEDIGO</span>
                                            </div>
                                            <p className="text-xs">{message.recommendation}</p>
                                        </div>
                                    )}
                                    
                                    <p className={`text-xs mt-3 ${
                                        message.sender === 'user' ? 'text-blue-100' : 'text-gray-500'
                                    }`}>
                                        {formatTime(message.timestamp)}
                                    </p>
                                </div>
                            </div>
                        ))}
                        
                        {isTyping && (
                            <div className="flex justify-start">
                                <div className="bg-white shadow-lg px-5 py-4 rounded-3xl border border-gray-100">
                                    <div className="flex items-center space-x-2">
                                        <span className="text-sm text-gray-600">MEDIGO réfléchit</span>
                                        <div className="flex space-x-1">
                                            <div className="w-2 h-2 bg-blue-400 rounded-full animate-bounce"></div>
                                            <div className="w-2 h-2 bg-purple-400 rounded-full animate-bounce" style={{animationDelay: '0.1s'}}></div>
                                            <div className="w-2 h-2 bg-pink-400 rounded-full animate-bounce" style={{animationDelay: '0.2s'}}></div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        )}
                        <div ref={messagesEndRef} />
                    </div>

                    <div className="p-4 bg-white/80 backdrop-blur-sm border-t border-gray-200">
                        <div className="flex space-x-3">
                            <input
                                type="text"
                                value={inputText}
                                onChange={(e) => setInputText(e.target.value)}
                                onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
                                placeholder="Raconte-moi ce qui ne va pas... 🤔"
                                className="flex-1 px-5 py-4 border-2 border-gray-200 rounded-full focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent bg-white shadow-lg text-sm"
                            />
                            <button
                                onClick={sendMessage}
                                disabled={!inputText.trim() || isTyping}
                                className="px-6 py-4 bg-gradient-to-r from-blue-500 to-purple-600 text-white rounded-full hover:from-blue-600 hover:to-purple-700 focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:opacity-50 disabled:cursor-not-allowed transition-all duration-300 shadow-lg transform hover:scale-105"
                            >
                                <Send className="w-5 h-5" />
                            </button>
                        </div>
                    </div>
                </div>
            );

            const DoctorsView = () => (
                <div className="p-6 space-y-6 bg-gradient-to-br from-green-50 to-blue-50 min-h-full">
                    <div className="text-center">
                        <div className="w-20 h-20 bg-gradient-to-br from-green-500 to-blue-600 rounded-full mx-auto mb-4 flex items-center justify-center shadow-xl">
                            <MapPin className="w-10 h-10 text-white" />
                        </div>
                        <h1 className="text-2xl font-bold text-gray-800 mb-2">Médecins près de toi</h1>
                        <p className="text-gray-600">📍 Paris, Île-de-France</p>
                    </div>

                    <div className="bg-gradient-to-r from-green-500 to-blue-600 rounded-2xl p-4 text-white shadow-xl">
                        <div className="flex items-center mb-2">
                            <Navigation className="w-5 h-5 mr-2" />
                            <span className="font-semibold">Géolocalisation activée</span>
                        </div>
                        <p className="text-green-100 text-sm">
                            Je recherche les meilleurs médecins dans ton secteur ! 🎯
                        </p>
                    </div>

                    <div className="space-y-4">
                        {nearbyDoctors.map((doctor) => (
                            <div key={doctor.id} className="bg-white rounded-2xl p-6 shadow-xl hover:shadow-2xl transition-all duration-300 transform hover:-translate-y-1">
                                <div className="flex items-start space-x-4">
                                    <div className="w-16 h-16 bg-gradient-to-br from-blue-400 to-purple-600 rounded-2xl flex items-center justify-center text-2xl shadow-lg">
                                        {doctor.avatar}
                                    </div>
                                    
                                    <div className="flex-1">
                                        <div className="flex items-center justify-between mb-2">
                                            <h3 className="font-bold text-gray-800 text-lg">{doctor.name}</h3>
                                            <div className="flex items-center space-x-1 bg-yellow-100 px-3 py-1 rounded-full">
                                                <Star className="w-4 h-4 text-yellow-500 fill-current" />
                                                <span className="text-sm font-semibold text-yellow-700">{doctor.rating}</span>
                                            </div>
                                        </div>
                                        
                                        <div className="space-y-2 text-sm">
                                            <div className="flex items-center space-x-2">
                                                <Stethoscope className="w-4 h-4 text-blue-500" />
                                                <span className="text-gray-600">{doctor.specialty}</span>
                                            </div>
                                            
                                            <div className="flex items-center space-x-2">
                                                <MapPin className="w-4 h-4 text-green-500" />
                                                <span className="text-gray-600">{doctor.distance} • {doctor.address}</span>
                                            </div>
                                            
                                            <div className="flex items-center space-x-2">
                                                <Clock className="w-4 h-4 text-purple-500" />
                                                <span className="text-gray-600">{doctor.nextAvailable}</span>
                                            </div>
                                        </div>
                                        
                                        <div className="flex space-x-3 mt-4">
                                            <button className="flex-1 bg-gradient-to-r from-blue-500 to-purple-600 text-white py-3 rounded-xl font-semibold hover:from-blue-600 hover:to-purple-700 transition-all duration-300 transform hover:scale-105 shadow-lg">
                                                📅 Prendre RDV
                                            </button>
                                            <button className="px-4 py-3 bg-green-500 text-white rounded-xl hover:bg-green-600 transition-colors shadow-lg">
                                                <Phone className="w-5 h-5" />
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        ))}
                    </div>
                </div>
            );

            const ProfileView = () => (
                <div className="p-6 space-y-6 bg-gradient-to-br from-purple-50 to-pink-50 min-h-full">
                    <div className="text-center">
                        <div className="w-24 h-24 bg-gradient-to-br from-purple-500 to-pink-600 rounded-full mx-auto mb-4 flex items-center justify-center shadow-xl">
                            <span className="text-3xl">👤</span>
                        </div>
                        <h2 className="text-2xl font-bold bg-gradient-to-r from-purple-600 to-pink-600 bg-clip-text text-transparent">Mon Profil</h2>
                        <p className="text-gray-600">Tes infos santé en un coup d'œil ! ✨</p>
                    </div>

                    <div className="bg-white rounded-2xl p-6 shadow-xl">
                        <div className="flex items-center mb-4">
                            <span className="text-xl mr-2">📊</span>
                            <h3 className="font-bold text-gray-800">Mes statistiques</h3>
                        </div>
                        <div className="grid grid-cols-2 gap-4">
                            <div className="bg-gradient-to-br from-blue-100 to-blue-200 p-4 rounded-xl text-center">
                                <div className="text-2xl font-bold text-blue-600">{messages.filter(m => m.sender === 'user').length}</div>
                                <div className="text-xs text-blue-700 font-medium">Questions posées</div>
                            </div>
                            <div className="bg-gradient-to-br from-green-100 to-green-200 p-4 rounded-xl text-center">
                                <div className="text-2xl font-bold text-green-600">3</div>
                                <div className="text-xs text-green-700 font-medium">Médecins trouvés</div>
                            </div>
                        </div>
                    </div>

                    <div className="bg-gradient-to-r from-yellow-100 to-orange-100 rounded-2xl p-6 border border-yellow-200 shadow-lg">
                        <div className="flex items-center mb-3">
                            <span className="text-2xl mr-2">🛡️</span>
                            <h3 className="font-bold text-yellow-800">Rappel important</h3>
                        </div>
                        <p className="text-yellow-700 text-sm">
                            Hey ! MEDIGO c'est super pour les petites questions, mais pour du sérieux, 
                            ton médecin reste le boss ! 👨‍⚕️✨ On se complète, on ne se remplace pas ! 😉
                        </p>
                    </div>
                </div>
            );

            return (
                <div className="max-w-sm mx-auto bg-gray-50 min-h-screen flex flex-col">
                    <div className="flex-1">
                        {currentView === 'home' && <HomeView />}
                        {currentView === 'chat' && <ChatView />}
                        {currentView === 'doctors' && <DoctorsView />}
                        {currentView === 'profile' && <ProfileView />}
                    </div>

                    <nav className="bg-white/90 backdrop-blur-sm border-t border-gray-200 p-4 shadow-lg">
                        <div className="flex justify-around">
                            {[
                                { id: 'home', icon: Home, label: 'Accueil', emoji: '🏠' },
                                { id: 'chat', icon: MessageCircle, label: 'Chat', emoji: '💬' },
                                { id: 'doctors', icon: MapPin, label: 'Médecins', emoji: '👩‍⚕️' },
                                { id: 'profile', icon: User, label: 'Profil', emoji: '👤' }
                            ].map(({ id, icon: Icon, label, emoji }) => (
                                <button
                                    key={id}
                                    onClick={() => setCurrentView(id)}
                                    className={`flex flex-col items-center space-y-1 px-3 py-2 rounded-2xl transition-all duration-300 transform hover:scale-105 ${
                                        currentView === id
                                            ? 'bg-gradient-to-r from-blue-500 to-purple-600 text-white shadow-lg'
                                            : 'text-gray-600 hover:text-blue-600 hover:bg-blue-50'
                                    }`}
                                >
                                    <div className="relative">
                                        <Icon className="w-6 h-6" />
                                        <span className="absolute -top-1 -right-1 text-xs">{emoji}</span>
                                    </div>
                                    <span className="text-xs font-medium">{label}</span>
                                </button>
                            ))}
                        </div>
                    </nav>
                </div>
            );
        };

        // PWA Installation Logic
        let deferredPrompt;
        const installPrompt = document.getElementById('install-prompt');
        const installBtn = document.getElementById('install-btn');
        const dismissBtn = document.getElementById('dismiss-btn');

        window.addEventListener('beforeinstallprompt', (e) => {
            e.preventDefault();
            deferredPrompt = e;
            installPrompt.style.display = 'block';
        });

        installBtn.addEventListener('click', async () => {
            if (deferredPrompt) {
                deferredPrompt.prompt();
                const { outcome } = await deferredPrompt.userChoice;
                deferredPrompt = null;
                installPrompt.style.display = 'none';
            }
        });

        dismissBtn.addEventListener('click', () => {
            installPrompt.style.display = 'none';
        });

        // Splash Screen Logic
        window.addEventListener('load', () => {
            setTimeout(() => {
                document.getElementById('splash').classList.add('fade-out');
                setTimeout(() => {
                    document.getElementById('splash').style.display = 'none';
                }, 500);
            }, 2000);
        });

        // Service Worker Registration
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                const swCode = `
                    const CACHE_NAME = 'medigo-v1';
                    const urlsToCache = [
                        '/',
                        'https://cdn.tailwindcss.com',
                        'https://unpkg.com/lucide@latest/dist/umd/lucide.js'
                    ];

                    self.addEventListener('install', (event) => {
                        event.waitUntil(
                            caches.open(CACHE_NAME)
                                .then((cache) => cache.addAll(urlsToCache))
                        );
                    });

                    self.addEventListener('fetch', (event) => {
                        event.respondWith(
                            caches.match(event.request)
                                .then((response) => response || fetch(event.request))
                        );
                    });
                `;
                
                const blob = new Blob([swCode], { type: 'application/javascript' });
                const swUrl = URL.createObjectURL(blob);
                
                navigator.serviceWorker.register(swUrl)
                    .then(() => console.log('SW registered'))
                    .catch(() => console.log('SW registration failed'));
            });
        }

        // Generate Manifest dynamically
        const manifest = {
            "name": "MEDIGO - Assistant Médical",
            "short_name": "MEDIGO",
            "description": "Votre assistant médical personnel - Questions santé, conseils et médecins à proximité",
            "start_url": "/",
            "display": "standalone",
            "background_color": "#ffffff",
            "theme_color": "#3B82F6",
            "icons": [
                {
                    "src": "data:image/svg+xml;base64," + btoa(`
                        <svg width="192" height="192" viewBox="0 0 192 192" fill="none" xmlns="http://www.w3.org/2000/svg">
                            <rect width="192" height="192" rx="32" fill="url(#gradient)"/>
                            <text x="96" y="120" font-size="80" text-anchor="middle" fill="white">🏥</text>
                            <defs>
                                <linearGradient id="gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                                    <stop offset="0%" style="stop-color:#3B82F6"/>
                                    <stop offset="100%" style="stop-color:#8B5CF6"/>
                                </linearGradient>
                            </defs>
                        </svg>
                    `),
                    "sizes": "192x192",
                    "type": "image/svg+xml"
                },
                {
                    "src": "data:image/svg+xml;base64," + btoa(`
                        <svg width="512" height="512" viewBox="0 0 512 512" fill="none" xmlns="http://www.w3.org/2000/svg">
                            <rect width="512" height="512" rx="64" fill="url(#gradient)"/>
                            <text x="256" y="320" font-size="200" text-anchor="middle" fill="white">🏥</text>
                            <defs>
                                <linearGradient id="gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                                    <stop offset="0%" style="stop-color:#3B82F6"/>
                                    <stop offset="100%" style="stop-color:#8B5CF6"/>
                                </linearGradient>
                            </defs>
                        </svg>
                    `),
                    "sizes": "512x512",
                    "type": "image/svg+xml"
                }
            ]
        };

        const manifestBlob = new Blob([JSON.stringify(manifest)], { type: 'application/json' });
        const manifestUrl = URL.createObjectURL(manifestBlob);
        document.getElementById('manifest-placeholder').href = manifestUrl;

        // Render App
        ReactDOM.render(<MedigoApp />, document.getElementById('app'));
    </script>
</body>
</html>
