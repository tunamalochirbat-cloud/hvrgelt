[index.html](https://github.com/user-attachments/files/24991991/index.html)
<!DOCTYPE html>
<html lang="mn">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Миний Карго - Хүргэлтийн систем</title>

    <!-- Icon -->
    <link rel="icon"
        href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2290%22>📦</text></svg>">

    <!-- Tailwind CSS (Загвар) -->
    <script src="https://cdn.tailwindcss.com"></script>

    <!-- React & Babel (Хөдөлгүүр) -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

    <!-- Lucide Icons (Дүрс) - Тогтвортой хувилбар 0.263.1 -->
    <script src="https://unpkg.com/lucide-react@0.263.1/dist/umd/lucide-react.min.js"></script>

    <style>
        /* Гүйлгэх мөрийг нуух (Scrollbar hide) */
        .no-scrollbar::-webkit-scrollbar {
            display: none;
        }

        .no-scrollbar {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }

        /* Modal animation */
        @keyframes slideUp {
            from {
                transform: translateY(100%);
                opacity: 0;
            }

            to {
                transform: translateY(0);
                opacity: 1;
            }
        }

        .animate-slide-up {
            animation: slideUp 0.3s ease-out forwards;
        }
    </style>
</head>

<body class="bg-slate-50">
    <div id="root"></div>

    <script type="text/babel">
        // React болон Lucide-г глобал хувьсагчаас авах
        const { useState, useEffect } = React;

        // Lucide icons-ийг алдаагүй дуудахын тулд шалгаж авна
        const {
            Package, Truck, MapPin, CheckCircle, Plus, Search,
            Copy, ExternalLink, Clipboard, Box, Facebook,
            Instagram, Globe, Trash2
        } = window.lucideReact || {};

        const App = () => {
            // User's Cargo Addresses
            const warehouses = [
                {
                    id: 1,
                    name: 'Монгол ложистик (Buhee)',
                    receiver: '白金成（buhee）',
                    phone: '134 2224 4242',
                    address: '广东省广州市荔湾区站前路96号蒙古物流',
                    note: 'Buhee'
                },
                {
                    id: 2,
                    name: '73 Монгол улс ложистик',
                    receiver: 'Contact Person',
                    phone: '13948713456',
                    address: '流花新街16号，73蒙古国物流',
                    note: '73 Cargo'
                }
            ];

            // LocalStorage-оос өгөгдөл унших
            const [orders, setOrders] = useState(() => {
                const savedOrders = localStorage.getItem('myCargoOrders');
                if (savedOrders) {
                    return JSON.parse(savedOrders);
                }
                return [
                    {
                        id: 'ORD-001',
                        itemName: 'Жишээ: Өвлийн гутал (Туршилт)',
                        trackingCode: 'YT2354891234',
                        markCode: '99887766',
                        warehouseId: 1,
                        status: 'arrived_ub',
                        date: '2023-10-25',
                        note: 'Хайрцагтай нь авах'
                    }
                ];
            });

            const [showForm, setShowForm] = useState(false);
            const [searchTerm, setSearchTerm] = useState('');
            const [showAddressModal, setShowAddressModal] = useState(false);

            // LocalStorage руу хадгалах
            useEffect(() => {
                localStorage.setItem('myCargoOrders', JSON.stringify(orders));
            }, [orders]);

            // New Order Form State
            const [newOrder, setNewOrder] = useState({
                itemName: '',
                trackingCode: '',
                markCode: '',
                warehouseId: 1,
                note: '',
                address: '',      // New field
                phoneNumber: ''   // New field
            });

            // Status definitions
            const statuses = {
                ordered: { label: 'Захиалсан', color: 'bg-gray-100 text-gray-800', icon: Clipboard, desc: 'Дэлгүүрээс код хүлээж байна' },
                china_warehouse: { label: 'Хятад агуулах', color: 'bg-yellow-100 text-yellow-800', icon: Box, desc: 'Карго дээр очсон' },
                transit: { label: 'Монгол руу гарсан', color: 'bg-blue-100 text-blue-800', icon: Truck, desc: 'Эрээн/Замын-Үүд' },
                arrived_ub: { label: 'УБ-д ирсэн', color: 'bg-green-100 text-green-800', icon: CheckCircle, desc: 'Утасдсан/Авах боломжтой' },
            };

            const handleAddOrder = (e) => {
                e.preventDefault();
                const id = `TRK-${Math.floor(Math.random() * 10000).toString().padStart(4, '0')}`;
                const order = {
                    id,
                    ...newOrder,
                    warehouseId: Number(newOrder.warehouseId),
                    status: 'ordered',
                    date: new Date().toISOString().split('T')[0],
                };
                setOrders([order, ...orders]);
                setNewOrder({
                    itemName: '',
                    trackingCode: '',
                    markCode: newOrder.markCode,
                    warehouseId: 1,
                    note: '',
                    address: '',      // Reset
                    phoneNumber: ''   // Reset
                });
                setShowForm(false);
            };

            const updateStatus = (id, newStatus) => {
                setOrders(orders.map(order => order.id === id ? { ...order, status: newStatus } : order));
            };

            const deleteOrder = (id) => {
                if (window.confirm('Энэ захиалгыг устгах уу?')) {
                    setOrders(orders.filter(o => o.id !== id));
                }
            };

            const copyToClipboard = (text) => {
                navigator.clipboard.writeText(text);
                alert("Хаяг хуулагдлаа: " + text);
            };

            const filteredOrders = orders.filter(order =>
                order.itemName.toLowerCase().includes(searchTerm.toLowerCase()) ||
                order.trackingCode.toLowerCase().includes(searchTerm.toLowerCase()) ||
                order.markCode.includes(searchTerm)
            );

            // Хэрэв Icon library ачааллаагүй бол алдаа заахгүйгээр render хийх
            if (!Package) return <div className="flex items-center justify-center h-screen">Loading icons...</div>;

            return (
                <div className="min-h-screen bg-slate-50 text-slate-800 font-sans pb-24">
                    {/* Header */}
                    <header className="bg-white shadow-sm sticky top-0 z-10 border-b border-slate-200">
                        <div className="max-w-3xl mx-auto px-4 py-4">
                            <div className="flex justify-between items-center mb-4">
                                <div className="flex items-center gap-2">
                                    <div className="bg-blue-600 p-2 rounded-lg">
                                        <Package className="w-6 h-6 text-white" />
                                    </div>
                                    <div>
                                        <h1 className="text-xl font-bold text-gray-900 leading-none">Миний Карго</h1>
                                        <span className="text-xs text-gray-500">Guangzhou &rarr; Ulaanbaatar</span>
                                    </div>
                                </div>
                                <button
                                    onClick={() => setShowAddressModal(true)}
                                    className="text-blue-600 bg-blue-50 px-3 py-2 rounded-lg text-sm font-medium hover:bg-blue-100 transition"
                                >
                                    Хаяг авах
                                </button>
                            </div>

                            {/* Search Bar */}
                            <div className="relative">
                                <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400 w-5 h-5" />
                                <input
                                    type="text"
                                    placeholder="Барааны нэр, Код эсвэл Марк..."
                                    className="w-full pl-10 pr-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 bg-slate-50"
                                    value={searchTerm}
                                    onChange={(e) => setSearchTerm(e.target.value)}
                                />
                            </div>
                        </div>
                    </header>

                    <main className="max-w-3xl mx-auto px-4 py-6 space-y-4">

                        {/* Quick Stats */}
                        <div className="flex gap-2 overflow-x-auto pb-2 no-scrollbar">
                            <div className="flex-shrink-0 bg-white p-3 rounded-xl border border-gray-200 min-w-[120px]">
                                <div className="text-xs text-gray-500">Нийт</div>
                                <div className="font-bold text-lg">{orders.length}</div>
                            </div>
                            <div className="flex-shrink-0 bg-white p-3 rounded-xl border border-gray-200 min-w-[120px]">
                                <div className="text-xs text-gray-500">Замд яваа</div>
                                <div className="font-bold text-lg text-blue-600">{orders.filter(o => o.status === 'transit').length}</div>
                            </div>
                            <div className="flex-shrink-0 bg-white p-3 rounded-xl border border-gray-200 min-w-[120px]">
                                <div className="text-xs text-gray-500">УБ-д ирсэн</div>
                                <div className="font-bold text-lg text-green-600">{orders.filter(o => o.status === 'arrived_ub').length}</div>
                            </div>
                        </div>

                        {/* Orders List */}
                        {filteredOrders.map(order => {
                            const StatusIcon = statuses[order.status].icon;
                            const warehouse = warehouses.find(w => w.id === order.warehouseId);

                            return (
                                <div key={order.id} className="bg-white rounded-xl shadow-sm border border-gray-200 overflow-hidden">
                                    <div className="p-4">
                                        <div className="flex justify-between items-start mb-3">
                                            <div>
                                                <h3 className="font-bold text-gray-900">{order.itemName}</h3>
                                                <div className="text-xs text-gray-500 mt-1 flex items-center gap-1">
                                                    <span className="bg-gray-100 px-1.5 py-0.5 rounded text-gray-600 font-mono">{order.trackingCode || 'Кодгүй'}</span>
                                                    {order.markCode && <span className="bg-yellow-100 px-1.5 py-0.5 rounded text-yellow-800 font-bold font-mono">#{order.markCode}</span>}
                                                </div>
                                            </div>
                                            <div className={`px-2 py-1 rounded-lg text-xs font-semibold flex items-center gap-1 ${statuses[order.status].color}`}>
                                                <StatusIcon className="w-3 h-3" />
                                                {statuses[order.status].label}
                                            </div>
                                        </div>

                                        <div className="flex items-center gap-2 text-xs text-gray-600 mb-2 bg-slate-50 p-2 rounded border border-slate-100">
                                            <MapPin className="w-3 h-3 text-blue-500" />
                                            <span>Ирэх карго: <strong>{warehouse?.name}</strong></span>
                                        </div>

                                        {(order.address || order.phoneNumber) && (
                                            <div className="text-xs text-gray-600 mb-3 space-y-1 bg-blue-50/50 p-2 rounded border border-blue-100">
                                                {order.address && (
                                                    <div className="flex items-start gap-2">
                                                        <MapPin className="w-3 h-3 text-gray-400 mt-0.5 flex-shrink-0" />
                                                        <span>{order.address}</span>
                                                    </div>
                                                )}
                                                {order.phoneNumber && (
                                                    <div className="flex items-center gap-2">
                                                        <div className="w-3 h-3 flex items-center justify-center text-gray-400">📞</div>
                                                        <span className="font-mono">{order.phoneNumber}</span>
                                                    </div>
                                                )}
                                            </div>
                                        )}

                                        {order.note && (
                                            <p className="text-xs text-gray-500 italic mb-3">"{order.note}"</p>
                                        )}

                                        {/* Status Timeline */}
                                        <div className="relative pt-2 pb-2 px-1">
                                            <div className="flex justify-between items-center relative z-10">
                                                {Object.keys(statuses).map((s, index) => {
                                                    const isActive = s === order.status;
                                                    const isPast = Object.keys(statuses).indexOf(order.status) >= index;

                                                    return (
                                                        <div
                                                            key={s}
                                                            onClick={() => updateStatus(order.id, s)}
                                                            className={`w-3 h-3 rounded-full border-2 cursor-pointer transition-all ${isPast ? 'bg-blue-600 border-blue-600' : 'bg-white border-gray-300'}`}
                                                            title={statuses[s].label}
                                                        />
                                                    );
                                                })}
                                            </div>
                                            <div className="absolute top-3.5 left-0 w-full h-0.5 bg-gray-200 -z-0">
                                                <div
                                                    className="h-full bg-blue-600 transition-all duration-300"
                                                    style={{
                                                        width: order.status === 'ordered' ? '0%' :
                                                            order.status === 'china_warehouse' ? '33%' :
                                                                order.status === 'transit' ? '66%' : '100%'
                                                    }}
                                                ></div>
                                            </div>
                                            <div className="mt-2 text-center text-xs font-medium text-blue-800">
                                                {statuses[order.status].desc}
                                            </div>
                                        </div>
                                    </div>

                                    <div className="bg-gray-50 px-4 py-2 border-t border-gray-100 flex justify-end">
                                        <button onClick={() => deleteOrder(order.id)} className="flex items-center gap-1 text-xs text-red-500 hover:text-red-700 font-medium">
                                            <Trash2 className="w-3 h-3" /> Устгах
                                        </button>
                                    </div>
                                </div>
                            );
                        })}

                        {filteredOrders.length === 0 && (
                            <div className="text-center py-12 text-gray-400">
                                <Package className="w-12 h-12 mx-auto mb-2 opacity-20" />
                                <p>Бүртгэл хоосон байна</p>
                            </div>
                        )}

                        {/* Social Media Footer */}
                        <div className="mt-8 pt-8 pb-4 border-t border-gray-200 text-center">
                            <p className="text-xs text-gray-500 mb-3 uppercase tracking-wide font-semibold">Холбоо барих</p>
                            <div className="flex flex-wrap justify-center gap-4">
                                <a href="https://www.facebook.com/profile.php?id=61575260079474" target="_blank" rel="noopener noreferrer" className="flex items-center gap-2 px-4 py-2 bg-white border border-gray-200 rounded-lg text-gray-700 hover:text-blue-600 hover:border-blue-200 transition-all shadow-sm">
                                    <Facebook className="w-5 h-5" />
                                    <span className="text-sm font-medium">Facebook</span>
                                </a>
                                <a href="https://www.instagram.com/uyanga_creator/" target="_blank" rel="noopener noreferrer" className="flex items-center gap-2 px-4 py-2 bg-white border border-gray-200 rounded-lg text-gray-700 hover:text-pink-600 hover:border-pink-200 transition-all shadow-sm">
                                    <Instagram className="w-5 h-5" />
                                    <span className="text-sm font-medium">Instagram</span>
                                </a>
                                <a href="https://google.com" target="_blank" rel="noopener noreferrer" className="flex items-center gap-2 px-4 py-2 bg-white border border-gray-200 rounded-lg text-gray-700 hover:text-green-600 hover:border-green-200 transition-all shadow-sm">
                                    <Globe className="w-5 h-5" />
                                    <span className="text-sm font-medium">Google</span>
                                </a>
                            </div>
                        </div>

                    </main>

                    {/* Floating Action Button */}
                    <button
                        onClick={() => setShowForm(true)}
                        className="fixed bottom-6 right-6 bg-blue-600 text-white w-14 h-14 rounded-full shadow-lg shadow-blue-300 flex items-center justify-center hover:bg-blue-700 transition-transform hover:scale-105 active:scale-95 z-20"
                    >
                        <Plus className="w-8 h-8" />
                    </button>

                    {/* Add Order Modal */}
                    {showForm && (
                        <div className="fixed inset-0 bg-black/60 flex items-end sm:items-center justify-center p-0 sm:p-4 z-50 backdrop-blur-sm">
                            <div className="bg-white rounded-t-2xl sm:rounded-2xl w-full max-w-md shadow-2xl p-6 animate-slide-up">
                                <div className="flex justify-between items-center mb-6">
                                    <h2 className="text-xl font-bold text-gray-900">Бараа бүртгэх</h2>
                                    <button onClick={() => setShowForm(false)} className="text-gray-400 hover:text-gray-600">
                                        <span className="text-2xl">&times;</span>
                                    </button>
                                </div>

                                <form onSubmit={handleAddOrder} className="space-y-4">
                                    <div>
                                        <label className="block text-xs font-bold text-gray-500 uppercase mb-1">Барааны нэр</label>
                                        <input
                                            required
                                            type="text"
                                            className="w-full px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 outline-none"
                                            placeholder="Жишээ: Өвлийн гутал"
                                            value={newOrder.itemName}
                                            onChange={(e) => setNewOrder({ ...newOrder, itemName: e.target.value })}
                                        />
                                    </div>

                                    <div className="grid grid-cols-2 gap-4">
                                        <div>
                                            <label className="block text-xs font-bold text-gray-500 uppercase mb-1">Tracking Код (Хятад)</label>
                                            <input
                                                type="text"
                                                className="w-full px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 outline-none font-mono text-sm"
                                                placeholder="SF1234..."
                                                value={newOrder.trackingCode}
                                                onChange={(e) => setNewOrder({ ...newOrder, trackingCode: e.target.value })}
                                            />
                                        </div>
                                        <div>
                                            <label className="block text-xs font-bold text-gray-500 uppercase mb-1">Марк (Mark)</label>
                                            <input
                                                required
                                                type="text"
                                                className="w-full px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 outline-none font-mono text-sm bg-yellow-50"
                                                placeholder="8 оронтой тоо"
                                                value={newOrder.markCode}
                                                onChange={(e) => setNewOrder({ ...newOrder, markCode: e.target.value })}
                                            />
                                        </div>
                                    </div>

                                    <div className="grid grid-cols-1 gap-4">
                                        <div>
                                            <label className="block text-xs font-bold text-gray-500 uppercase mb-1">Хүргүүлэх Хаяг (Монголд)</label>
                                            <input
                                                type="text"
                                                className="w-full px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 outline-none text-sm"
                                                placeholder="Дүүрэг, Хороо, Байр, Орц..."
                                                value={newOrder.address}
                                                onChange={(e) => setNewOrder({ ...newOrder, address: e.target.value })}
                                            />
                                        </div>
                                        <div>
                                            <label className="block text-xs font-bold text-gray-500 uppercase mb-1">Утасны дугаар</label>
                                            <input
                                                type="tel"
                                                className="w-full px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 outline-none font-mono text-sm"
                                                placeholder="8888-8888"
                                                value={newOrder.phoneNumber}
                                                onChange={(e) => setNewOrder({ ...newOrder, phoneNumber: e.target.value })}
                                            />
                                        </div>
                                    </div>

                                    <div>
                                        <label className="block text-xs font-bold text-gray-500 uppercase mb-1">Карго сонгох</label>
                                        <select
                                            className="w-full px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 outline-none bg-white"
                                            value={newOrder.warehouseId}
                                            onChange={(e) => setNewOrder({ ...newOrder, warehouseId: e.target.value })}
                                        >
                                            {warehouses.map(w => (
                                                <option key={w.id} value={w.id}>{w.name}</option>
                                            ))}
                                        </select>
                                    </div>

                                    <div>
                                        <label className="block text-xs font-bold text-gray-500 uppercase mb-1">Тэмдэглэл (Сонголтоор)</label>
                                        <textarea
                                            className="w-full px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 outline-none h-20 resize-none"
                                            placeholder="Жишээ: Хайрцаггүй ирвэл буцаана..."
                                            value={newOrder.note}
                                            onChange={(e) => setNewOrder({ ...newOrder, note: e.target.value })}
                                        />
                                    </div>

                                    <button type="submit" className="w-full bg-blue-600 text-white py-4 rounded-xl font-bold hover:bg-blue-700 transition-colors">
                                        Хадгалах
                                    </button>
                                </form>
                            </div>
                        </div>
                    )}

                    {/* Address Info Modal */}
                    {showAddressModal && (
                        <div className="fixed inset-0 bg-black/60 flex items-center justify-center p-4 z-50 backdrop-blur-sm">
                            <div className="bg-white rounded-2xl w-full max-w-lg shadow-2xl p-6">
                                <div className="flex justify-between items-center mb-6">
                                    <h2 className="text-xl font-bold text-gray-900">Карго Хаягууд (Guangzhou)</h2>
                                    <button onClick={() => setShowAddressModal(false)} className="bg-gray-100 p-1 rounded-full hover:bg-gray-200">
                                        <span className="text-2xl px-2">&times;</span>
                                    </button>
                                </div>

                                <div className="space-y-4">
                                    <div className="p-3 bg-yellow-50 text-yellow-800 text-sm rounded-lg border border-yellow-200 mb-4">
                                        <span className="font-bold">Санамж:</span> "包装外必须写客户的8位数的唛头" (Хайрцагны гадна талд үйлчлүүлэгчийн 8 оронтой Марк кодоо заавал бичүүлээрэй!)
                                    </div>

                                    {warehouses.map(w => (
                                        <div key={w.id} className="border border-gray-200 rounded-xl p-4 hover:border-blue-300 transition-colors">
                                            <div className="flex justify-between items-start mb-2">
                                                <h3 className="font-bold text-lg text-blue-900">{w.name}</h3>
                                                <span className="text-xs bg-gray-100 px-2 py-1 rounded text-gray-600">Guangzhou</span>
                                            </div>

                                            <div className="space-y-3 text-sm">
                                                <div className="flex justify-between items-center bg-slate-50 p-2 rounded">
                                                    <span className="text-gray-500">Address:</span>
                                                    <div className="flex items-center gap-2">
                                                        <span className="font-medium text-gray-900 truncate max-w-[180px]">{w.address}</span>
                                                        <button onClick={() => copyToClipboard(w.address)} className="text-blue-600 hover:text-blue-800">
                                                            <Copy className="w-4 h-4" />
                                                        </button>
                                                    </div>
                                                </div>

                                                <div className="flex justify-between items-center bg-slate-50 p-2 rounded">
                                                    <span className="text-gray-500">Receiver:</span>
                                                    <div className="flex items-center gap-2">
                                                        <span className="font-medium text-gray-900">{w.receiver}</span>
                                                        <button onClick={() => copyToClipboard(w.receiver)} className="text-blue-600 hover:text-blue-800">
                                                            <Copy className="w-4 h-4" />
                                                        </button>
                                                    </div>
                                                </div>

                                                <div className="flex justify-between items-center bg-slate-50 p-2 rounded">
                                                    <span className="text-gray-500">Phone:</span>
                                                    <div className="flex items-center gap-2">
                                                        <span className="font-medium text-gray-900">{w.phone}</span>
                                                        <button onClick={() => copyToClipboard(w.phone)} className="text-blue-600 hover:text-blue-800">
                                                            <Copy className="w-4 h-4" />
                                                        </button>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                    ))}
                                </div>

                                <button onClick={() => setShowAddressModal(false)} className="w-full mt-6 bg-gray-100 text-gray-700 py-3 rounded-xl font-semibold hover:bg-gray-200">
                                    Хаах
                                </button>
                            </div>
                        </div>
                    )}
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>

</html>
