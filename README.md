import React, { useState, useEffect } from 'react';
import { ShoppingCart, Hammer, Award, Shield, Phone, ChevronRight, X, Music, MapPin, CheckCircle2, Star, ExternalLink, Heart, Plus, Loader2, Settings, Trash2, Save } from 'lucide-react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from 'firebase/auth';
import { getFirestore, collection, onSnapshot, query, addDoc, deleteDoc, doc, serverTimestamp } from 'firebase/firestore';

// Firebase 配置
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

const App = () => {
  const [currentPage, setCurrentPage] = useState('home');
  const [user, setUser] = useState(null);
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [showAdmin, setShowAdmin] = useState(false); // 管理員模式開關
  const [adminClickCount, setAdminClickCount] = useState(0);

  // 1. 處理身份驗證
  useEffect(() => {
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (error) {
        console.error("Auth error:", error);
      }
    };
    initAuth();
    const unsubscribe = onAuthStateChanged(auth, setUser);
    return () => unsubscribe();
  }, []);

  // 2. 監聽商品資料即時同步
  useEffect(() => {
    if (!user) return;
    const productsRef = collection(db, 'artifacts', appId, 'public', 'data', 'products');
    const unsubscribe = onSnapshot(productsRef, (snapshot) => {
      const productList = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setProducts(productList.sort((a, b) => (b.createdAt?.seconds || 0) - (a.createdAt?.seconds || 0)));
      setLoading(false);
    }, (error) => {
      console.error("Firestore Error:", error);
      setLoading(false);
    });
    return () => unsubscribe();
  }, [user]);

  // 觸發隱藏管理模式
  const handleAdminTrigger = () => {
    const newCount = adminClickCount + 1;
    setAdminClickCount(newCount);
    if (newCount >= 3) {
      setShowAdmin(true);
      setAdminClickCount(0);
    }
  };

  return (
    <div className="min-h-screen bg-stone-50 text-stone-900 font-sans selection:bg-amber-200">
      {/* 導航欄 */}
      <nav className="fixed w-full z-50 bg-stone-900/95 text-white backdrop-blur-md border-b border-stone-800">
        <div className="max-w-7xl mx-auto px-6 h-20 flex justify-between items-center">
          <div className="flex items-center gap-4 cursor-pointer" onClick={() => setCurrentPage('home')}>
            <img src="https://i.imgur.com/H7NWyZP.png" alt="TK Logo" className="h-10 w-auto" />
            <div className="flex flex-col border-l border-stone-700 pl-4">
              <span className="text-xl font-bold tracking-widest text-amber-500 italic leading-none text-nowrap uppercase">TK SAXOPHONE</span>
              <span className="text-[9px] tracking-[0.2em] text-stone-400 font-bold uppercase mt-1 text-nowrap">傳承三代 | 100% 台灣製造</span>
            </div>
          </div>
          
          <div className="hidden md:flex space-x-10 text-xs font-bold tracking-widest uppercase">
            <button onClick={() => setCurrentPage('home')} className={currentPage === 'home' ? 'text-amber-500' : 'hover:text-amber-400 transition'}>首頁</button>
            <button onClick={() => setCurrentPage('about')} className={currentPage === 'about' ? 'text-amber-500' : 'hover:text-amber-400 transition'}>品牌簡介</button>
            <button onClick={() => setCurrentPage('products')} className={currentPage === 'products' ? 'text-amber-500' : 'hover:text-amber-400 transition'}>產品系列</button>
            <button onClick={() => setCurrentPage('service')} className={currentPage === 'service' ? 'text-amber-500' : 'hover:text-amber-400 transition'}>售後服務</button>
          </div>

          <div className="flex items-center gap-4">
            <a href="https://t27v5glqel.cashier.ecpay.com.tw" target="_blank" rel="noopener noreferrer" className="p-2 hover:bg-stone-800 rounded-full transition flex items-center gap-2 text-xs font-bold">
              <ShoppingCart size={22} className="text-amber-500" />
              <span className="hidden sm:inline uppercase tracking-widest text-white">進入綠界商城</span>
            </a>
            {showAdmin && (
              <button onClick={() => setCurrentPage('admin')} className="p-2 bg-amber-600 rounded-full animate-pulse"><Settings size={20} /></button>
            )}
          </div>
        </div>
      </nav>

      <main className="pt-20">
        {currentPage === 'home' && <HomePage onExplore={() => setCurrentPage('products')} />}
        {currentPage === 'about' && <AboutSection />}
        {currentPage === 'products' && <ProductsPage products={products} loading={loading} />}
        {currentPage === 'service' && <ServicePage />}
        {currentPage === 'admin' && <AdminPanel products={products} onClose={() => setCurrentPage('products')} />}
      </main>

      {/* 頁腳 */}
      <footer className="bg-stone-950 text-stone-500 py-16 text-left">
        <div className="max-w-7xl mx-auto px-6 grid md:grid-cols-4 gap-12">
          <div className="col-span-2">
            <div className="flex items-center gap-4 mb-6">
               <img src="https://i.imgur.com/H7NWyZP.png" alt="Logo" className="h-8 opacity-90" />
               <span className="text-xl font-bold tracking-widest text-white italic text-nowrap uppercase">TK SAXOPHONE</span>
            </div>
            <p className="text-sm leading-relaxed max-w-sm">
              八十年，我們只專心做好一件事：製作能與靈魂共鳴的薩克斯風。堅持 100% 台灣製造，守護職人工藝。
            </p>
          </div>
          <div>
            <h4 className="text-white font-bold text-sm tracking-widest uppercase mb-6">聯繫資訊</h4>
            <div className="space-y-4 text-xs">
              <p className="flex items-start gap-2 leading-relaxed font-bold text-stone-300 italic underline underline-offset-4 decoration-amber-500">0911-375938 Easen</p>
              <p className="flex items-start gap-2 leading-relaxed"><MapPin size={16} className="text-amber-500 flex-shrink-0" /> 台中旗艦店</p>
            </div>
          </div>
          <div>
            <h4 onClick={handleAdminTrigger} className="text-white font-bold text-sm tracking-widest uppercase mb-6 cursor-default select-none">購物保障</h4>
            <div className="space-y-3">
              <div className="flex items-center gap-2 text-[10px] text-green-500 uppercase font-bold tracking-widest italic">
                <div className="w-2 h-2 bg-green-500 rounded-full animate-pulse"></div>
                綠界支付安全對接中
              </div>
              <p className="text-[10px] font-bold text-stone-600">ECPay 安全金流認證</p>
            </div>
          </div>
        </div>
        <div className="border-t border-stone-900 max-w-7xl mx-auto mt-16 pt-8 text-center text-[9px] tracking-widest uppercase text-stone-700 italic">
          © 2024 TK MELODY SAXOPHONE. CLOUD SYNCED INTERFACE.
        </div>
      </footer>
    </div>
  );
};

// 管理員控制面板組件
const AdminPanel = ({ products, onClose }) => {
  const [newProduct, setNewProduct] = useState({ name: '', price: '', img: '', url: '' });
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleAdd = async (e) => {
    e.preventDefault();
    setIsSubmitting(true);
    try {
      const productsRef = collection(db, 'artifacts', appId, 'public', 'data', 'products');
      await addDoc(productsRef, {
        ...newProduct,
        price: Number(newProduct.price),
        createdAt: serverTimestamp()
      });
      setNewProduct({ name: '', price: '', img: '', url: '' });
      alert("商品已同步上架！");
    } catch (err) {
      alert("上架失敗，請檢查權限設定");
    } finally {
      setIsSubmitting(false);
    }
  };

  const handleDelete = async (id) => {
    if (!window.confirm("確定要刪除此商品嗎？")) return;
    try {
      await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'products', id));
    } catch (err) {
      alert("刪除失敗");
    }
  };

  return (
    <div className="max-w-4xl mx-auto px-6 py-20 animate-in slide-in-from-bottom-4 duration-500">
      <div className="flex justify-between items-center mb-12 border-b border-stone-200 pb-6">
        <h2 className="text-3xl font-black italic flex items-center gap-3">
          <Settings className="text-amber-600" /> 商品同步管理後台
        </h2>
        <button onClick={onClose} className="text-sm font-bold text-stone-400 hover:text-stone-900">返回產品頁</button>
      </div>

      {/* 新增商品表單 */}
      <div className="bg-white p-8 rounded-3xl border border-stone-200 shadow-sm mb-12">
        <h3 className="text-lg font-bold mb-6 italic underline underline-offset-4 decoration-amber-500">上架新商品 (綠界同步)</h3>
        <form onSubmit={handleAdd} className="grid md:grid-cols-2 gap-6">
          <div className="space-y-2">
            <label className="text-[10px] font-bold text-stone-400 uppercase tracking-widest">商品名稱</label>
            <input required value={newProduct.name} onChange={e => setNewProduct({...newProduct, name: e.target.value})} className="w-full bg-stone-50 border border-stone-100 p-3 rounded-xl focus:ring-2 focus:ring-amber-500 outline-none text-sm" placeholder="例如: TK Legend ALTO SAX" />
          </div>
          <div className="space-y-2">
            <label className="text-[10px] font-bold text-stone-400 uppercase tracking-widest">商品價格 (數字)</label>
            <input required type="number" value={newProduct.price} onChange={e => setNewProduct({...newProduct, price: e.target.value})} className="w-full bg-stone-50 border border-stone-100 p-3 rounded-xl focus:ring-2 focus:ring-amber-500 outline-none text-sm" placeholder="72000" />
          </div>
          <div className="space-y-2">
            <label className="text-[10px] font-bold text-stone-400 uppercase tracking-widest">綠界商品連結 (ECPay URL)</label>
            <input required value={newProduct.url} onChange={e => setNewProduct({...newProduct, url: e.target.value})} className="w-full bg-stone-50 border border-stone-100 p-3 rounded-xl focus:ring-2 focus:ring-amber-500 outline-none text-sm" placeholder="https://t27v5glqel.cashier..." />
          </div>
          <div className="space-y-2">
            <label className="text-[10px] font-bold text-stone-400 uppercase tracking-widest">照片 URL (Imgur 連結)</label>
            <input required value={newProduct.img} onChange={e => setNewProduct({...newProduct, img: e.target.value})} className="w-full bg-stone-50 border border-stone-100 p-3 rounded-xl focus:ring-2 focus:ring-amber-500 outline-none text-sm" placeholder="https://i.imgur.com/..." />
          </div>
          <button disabled={isSubmitting} className="md:col-span-2 py-4 bg-stone-900 text-white rounded-2xl font-black hover:bg-amber-600 transition flex items-center justify-center gap-3">
            {isSubmitting ? <Loader2 className="animate-spin" /> : <Save size={18} />} 立即同步至網頁
          </button>
        </form>
      </div>

      {/* 現有商品清單 */}
      <div className="space-y-4">
        <h3 className="text-lg font-bold mb-4 italic">目前線上商品 ({products.length})</h3>
        {products.map(p => (
          <div key={p.id} className="bg-white p-4 rounded-2xl border border-stone-100 flex items-center justify-between group">
            <div className="flex items-center gap-4 text-left">
              <img src={p.img} className="w-12 h-12 rounded object-cover" />
              <div>
                <h4 className="font-bold text-sm">{p.name}</h4>
                <p className="text-xs text-amber-700 font-black">NT$ {Number(p.price).toLocaleString()}</p>
              </div>
            </div>
            <button onClick={() => handleDelete(p.id)} className="p-2 text-stone-200 hover:text-red-500 transition"><Trash2 size={18} /></button>
          </div>
        ))}
      </div>
    </div>
  );
};

// 首頁組件
const HomePage = ({ onExplore }) => (
  <div className="animate-in fade-in duration-1000">
    <section className="relative h-[90vh] flex items-center overflow-hidden">
      <div className="absolute inset-0">
        <img src="https://i.imgur.com/3jRKwrH.jpeg" alt="背景" className="w-full h-full object-cover" />
        <div className="absolute inset-0 bg-stone-950/40"></div>
      </div>
      <div className="relative max-w-7xl mx-auto px-6 w-full text-center md:text-left">
        <div className="max-w-4xl space-y-8">
          <div className="inline-block py-1 px-3 bg-amber-600 text-white text-[10px] font-black tracking-[0.4em] uppercase rounded shadow-lg">Since 1946 Legacy</div>
          <h1 className="text-6xl md:text-8xl font-black text-white leading-[1.1] tracking-tighter italic drop-shadow-2xl uppercase">
            T.K. SAXOPHONE<br /><span className="text-amber-500 text-4xl md:text-6xl block mt-4">傳承八十載的台灣匠心</span>
          </h1>
          <p className="text-xl text-stone-100 font-light max-w-2xl leading-relaxed drop-shadow-md mx-auto md:mx-0">
            「八十年，我們只專心做好一件事：製作能與靈魂共鳴的薩克斯風。」
          </p>
          <div className="flex justify-center md:justify-start gap-6 pt-4">
            <button onClick={onExplore} className="px-10 py-5 bg-amber-600 text-white rounded-full font-black hover:bg-amber-700 transition shadow-2xl uppercase tracking-widest text-xs flex items-center gap-2">
              前往旗艦商城 <ChevronRight size={18} />
            </button>
          </div>
        </div>
      </div>
    </section>

    {/* 代言人區塊 */}
    <section className="py-24 bg-stone-900 text-white overflow-hidden text-left">
      <div className="max-w-7xl mx-auto px-6">
        <div className="text-center mb-20">
          <h2 className="text-4xl font-black italic mb-4 tracking-tight">國際演奏家之選</h2>
          <p className="text-stone-500 text-sm tracking-[0.2em] uppercase italic">Trusted by World-Class Artists</p>
        </div>
        <div className="grid grid-cols-1 md:grid-cols-2 gap-12">
          {/* 陳嘉俊 */}
          <div className="relative group bg-stone-800/50 rounded-[40px] overflow-hidden flex flex-col shadow-2xl">
            <div className="h-[500px] w-full bg-stone-900">
              <img src="https://i.imgur.com/QsOg9vX.jpeg" alt="陳嘉俊" className="w-full h-full object-contain object-bottom transition duration-1000 group-hover:scale-105" />
            </div>
            <div className="absolute inset-x-0 bottom-0 p-10 bg-gradient-to-t from-stone-950 via-stone-950/60 to-transparent">
              <span className="text-[10px] font-black text-amber-500 tracking-[0.4em] uppercase mb-3 block italic">Official Endorser</span>
              <h4 className="text-4xl font-black mb-4 italic">陳嘉俊 (Wilson Chen)</h4>
              <p className="text-stone-300 font-light max-w-sm">其個人專屬「嘉」Model 系列在全球爵士圈享有盛譽。</p>
            </div>
          </div>
          {/* 小林香織 */}
          <div className="relative group bg-stone-800/50 rounded-[40px] overflow-hidden flex flex-col shadow-2xl">
            <div className="h-[500px] w-full bg-stone-900">
              <img src="https://i.imgur.com/MgniwDh.jpeg" alt="小林香織" className="w-full h-full object-contain object-bottom transition duration-1000 group-hover:scale-105" />
            </div>
            <div className="absolute inset-x-0 bottom-0 p-10 bg-gradient-to-t from-stone-950 via-stone-950/60 to-transparent">
              <span className="text-[10px] font-black text-amber-500 tracking-[0.4em] uppercase mb-3 block italic">Saxophone Queen</span>
              <h4 className="text-4xl font-black mb-4 italic">小林香織 (Kaori Kobayashi)</h4>
              <p className="text-stone-300 font-light max-w-sm">多次巡迴演出指定使用 TK 系列產品，體現美感的結合。</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    {/* 工藝細節 */}
    <section className="py-24 bg-white">
      <div className="max-w-7xl mx-auto px-6 text-center mb-16">
        <h2 className="text-4xl font-black italic mb-2 tracking-tight">精緻工藝細節</h2>
        <div className="w-16 h-1 bg-amber-500 mx-auto mt-4"></div>
      </div>
      <div className="max-w-5xl mx-auto px-6 grid grid-cols-1 md:grid-cols-2 gap-8">
        {["https://i.imgur.com/7h8Tm2w.jpeg", "https://i.imgur.com/ub1b35l.jpeg", "https://i.imgur.com/aN0jyY4.jpeg", "https://i.imgur.com/mOrQ8ou.jpeg"].map((img, i) => (
          <div key={i} className="aspect-[4/3] rounded-3xl overflow-hidden shadow-lg border border-stone-100 group">
            <img src={img} className="w-full h-full object-cover group-hover:scale-110 transition duration-700" alt="Detail" />
          </div>
        ))}
      </div>
    </section>
  </div>
);

const ProductsPage = ({ products, loading }) => {
  // 基礎備份數據
  const displayProducts = products.length > 0 ? products : [
    { name: 'TK.SAXOPHONE 傳奇中音 ALTO SAX', price: 72000, img: 'https://i.imgur.com/f3sTY2J.jpeg', url: 'https://t27v5glqel.cashier.ecpay.com.tw/product/000000000908931' },
    { name: 'TK.Saxophone 雅柏次中音 Tenor Sax', price: 120000, img: 'https://i.imgur.com/syUBobO.jpeg', url: 'https://t27v5glqel.cashier.ecpay.com.tw/product/000000000908930' }
  ];

  return (
    <div className="max-w-7xl mx-auto px-6 py-20 animate-in fade-in duration-700 text-left">
      <div className="text-center mb-16">
        <h2 className="text-4xl font-black italic mb-4 tracking-tight">旗艦店系列商品</h2>
        <p className="text-stone-400 font-light italic">與綠界科技金流同步。享受最安全的線上樂器選購體驗。</p>
      </div>

      {loading && products.length === 0 ? (
        <div className="flex flex-col items-center justify-center py-24 space-y-4">
          <Loader2 className="animate-spin text-amber-500" size={48} />
          <p className="text-stone-400 text-xs font-bold tracking-widest uppercase italic">雲端同步中...</p>
        </div>
      ) : (
        <div className="grid md:grid-cols-2 lg:grid-cols-4 gap-8">
          {displayProducts.map((product, idx) => (
            <div key={product.id || idx} className="bg-white rounded-3xl overflow-hidden border border-stone-100 shadow-sm hover:shadow-xl transition flex flex-col h-full group">
              <div className="aspect-square bg-stone-50 overflow-hidden">
                <img src={product.img} className="w-full h-full object-cover group-hover:scale-110 transition duration-700" alt={product.name} />
              </div>
              <div className="p-6 flex flex-col flex-1">
                <h3 className="font-bold text-sm mb-2 h-10 line-clamp-2">{product.name}</h3>
                <p className="text-amber-700 font-black text-xl mb-6 tracking-tight italic">NT$ {Number(product.price).toLocaleString()}</p>
                <a href={product.url} target="_blank" rel="noopener noreferrer" className="mt-auto w-full py-3 bg-stone-900 text-white rounded-xl text-[10px] font-bold tracking-widest uppercase hover:bg-amber-600 transition flex items-center justify-center gap-2">
                  綠界安全購買 <ExternalLink size={14} />
                </a>
              </div>
            </div>
          ))}
        </div>
      )}
    </div>
  );
};

const AboutSection = () => (
  <div className="max-w-5xl mx-auto px-6 py-24 animate-in fade-in text-left">
    <div className="text-center mb-16 space-y-4">
      <h2 className="text-5xl font-black italic mb-4 tracking-tight uppercase">T.K. SAXOPHONE</h2>
      <p className="text-amber-600 font-bold tracking-widest text-lg italic underline underline-offset-8">傳承八十載的台灣匠心</p>
    </div>
    <div className="prose prose-stone max-w-none space-y-12">
      <div className="bg-white p-10 rounded-[40px] border border-stone-100 shadow-sm text-center">
        <p className="text-2xl font-light leading-relaxed text-stone-800 italic">
          「八十年，我們只專心做好一件事：製作能與靈魂共鳴的薩克斯風。」
        </p>
      </div>
      <div className="grid md:grid-cols-2 gap-12 items-center text-left font-light leading-relaxed text-stone-600">
        <div className="space-y-6">
          <p className="text-lg leading-relaxed">
            來自台灣的 <span className="font-bold text-stone-900 underline decoration-amber-500 decoration-2 italic uppercase">T.K. SAXOPHONE</span>，擁有超過 <span className="font-bold text-stone-900 text-xl italic underline">80 年</span> 的專業製造底蘊。我們不僅深耕在地，更堅持 <span className="font-bold text-amber-700 italic">100% 台灣製造與銷售</span>，將每一把樂器視為「感性」與「創造性」的完美結合。
          </p>
        </div>
        <img src="https://i.imgur.com/DTuYgcN.jpeg" className="rounded-3xl shadow-lg" alt="TK 工藝" />
      </div>
    </div>
  </div>
);

const ServicePage = () => (
  <div className="max-w-4xl mx-auto px-6 py-20 text-center">
    <Shield size={48} className="mx-auto text-amber-600 mb-6" />
    <h2 className="text-4xl font-black italic mb-6">售後服務與保養</h2>
    <div className="bg-stone-900 text-white p-12 rounded-[40px] shadow-2xl">
      <p className="text-2xl font-black mb-4 tracking-widest italic tracking-tighter text-nowrap">0911-375938</p>
      <p className="text-stone-400 font-light mb-8 uppercase tracking-widest text-xs">專人服務：Easen</p>
      <button className="px-8 py-3 bg-amber-600 rounded-full font-bold hover:bg-amber-700 transition tracking-widest uppercase text-xs">預約維修校正</button>
    </div>
  </div>
);

export default App;
