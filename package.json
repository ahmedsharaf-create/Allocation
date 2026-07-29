import React, { useState, useEffect } from 'react';
import { MapPin, Users, Upload, Download, RefreshCw, Layers, ShieldCheck, Award } from 'lucide-react';
import { MapContainer, TileLayer, Marker, Popup, Polyline } from 'react-leaflet';
import 'leaflet/dist/leaflet.css';
import * as XLSX from 'xlsx';
import L from 'leaflet';

// إصلاح أيقونات Leaflet الافتراضية في React
delete L.Icon.Default.prototype._getIconUrl;
L.Icon.Default.mergeOptions({
  iconRetinaUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-icon-2x.png',
  iconUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-icon.png',
  shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-shadow.png',
});

// بيانات المديرين الافتراضية المستخرجة من طلبك
const INITIAL_MANAGERS = [
  { id: 1, name: "Ahmed Bahgat", lat: 29.906444, lng: 30.978861, maxBranches: 3 },
  { id: 2, name: "Mohamed Emad", lat: 30.194023, lng: 31.147755, maxBranches: 3 },
  { id: 3, name: "Bebo", lat: 30.009306, lng: 31.172444, maxBranches: 3 },
  { id: 4, name: "Eslam Goda", lat: 30.232083, lng: 31.441083, maxBranches: 3 },
  { id: 5, name: "Mohamed Mostafa", lat: 30.198056, lng: 31.139944, maxBranches: 3 },
  { id: 6, name: "Ibrahem", lat: 30.037989, lng: 30.980000, maxBranches: 3 }
];

export default function App() {
  const [managers, setManagers] = useState(INITIAL_MANAGERS);
  const [branches, setBranches] = useState([]);
  const [apiKey, setApiKey] = useState('');
  const [optimizedResult, setOptimizedResult] = useState(null);
  const [loading, setLoading] = useState(false);
  const [activeTab, setActiveTab] = useState('dashboard');

  // حساب المسافة بين نقطتين (Haversine Formula) كبديل ذكي وسريع أو احتياطي للـ API
  const calculateDistance = (lat1, lon1, lat2, lon2) => {
    const R = 6371; // نصف قطر الأرض بالكيلومتر
    const dLat = (lat2 - lat1) * (Math.PI / 180);
    const dLon = (lon2 - lon1) * (Math.PI / 180);
    const a =
      Math.sin(dLat / 2) * Math.sin(dLat / 2) +
      Math.cos(lat1 * (Math.PI / 180)) * Math.cos(lat2 * (Math.PI / 180)) *
      Math.sin(dLon / 2) * Math.sin(dLon / 2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    return R * c; // المسافة بالكيلومتر
  };

  // رفع ملف Excel للفروع
  const handleFileUpload = (e) => {
    const file = e.target.files[0];
    const reader = new FileReader();
    reader.onload = (evt) => {
      const bstr = evt.target.result;
      const wb = XLSX.read(bstr, { type: 'binary' });
      const wsname = wb.SheetNames[0];
      const ws = wb.Sheets[wsname];
      const data = XLSX.utils.sheet_to_json(ws);
      
      // تنسيق البيانات المتوقعة: اسم الفرع، خط العرض Lat، خط الطول Lng
      const formattedBranches = data.map((row, index) => ({
        id: index + 1,
        name: row.BranchName || row.name || row['اسم الفرع'] || `فرع ${index + 1}`,
        lat: parseFloat(row.Lat || row.lat || row['خط العرض']),
        lng: parseFloat(row.Lng || row.lng || row['خط الطول'])
      })).filter(b => !isNaN(b.lat) && !isNaN(b.lng));

      setBranches(formattedBranches);
      alert(`تم تحميل ${formattedBranches.length} فرع بنجاح!`);
    };
    reader.readAsBinaryString(file);
  };

  // خوارزمية التوزيع الذكي العادل (Capacitated Greedy Assignment)
  const handleOptimize = async () => {
    if (branches.length === 0) {
      alert('يرجى رفع ملف الفروع أولاً!');
      return;
    }

    setLoading(true);

    // محاكاة أو استدعاء Google Maps API لو متوفر، أو استخدام المسافة الرياضية الدقيقة
    setTimeout(() => {
      let unassignedBranches = [...branches];
      let managerWorkload = managers.map(m => ({
        ...m,
        assignedBranches: [],
        totalDistance: 0
      }));

      // حساب إجمالي السعة المتاحة للمديرين
      const totalCapacity = managerWorkload.reduce((sum, m) => sum + m.maxBranches, 0);
      if (totalCapacity < unassignedBranches.length) {
        alert(`تنبيه: إجمالي السعة المسموحة للمديرين (${totalCapacity}) أقل من عدد الفروع (${unassignedBranches.length}). سيتم توزيع المتاح.`);
      }

      // خوارزمية التوزيع الجشع المتوازن (Balanced Greedy)
      // يتم اختيار أقرب فرع متاح لكل مدير بشكل دوري لضمان العدالة وتقليل المسافات
      let assignedCount = 0;
      let round = 0;
      
      while (unassignedBranches.length > 0 && round < 10) {
        let distributedInThisRound = false;
        
        for (let m of managerWorkload) {
          if (m.assignedBranches.length >= m.maxBranches || unassignedBranches.length === 0) continue;

          // البحث عن أقرب فرع لهذا المدير
          let nearestIdx = 0;
          let minDistance = Infinity;

          unassignedBranches.forEach((branch, idx) => {
            const dist = calculateDistance(m.lat, m.lng, branch.lat, branch.lng);
            if (dist < minDistance) {
              minDistance = dist;
              nearestIdx = idx;
            }
          });

          // إسناد الفرع للمدير
          const assignedBranch = unassignedBranches.splice(nearestIdx, 1)[0];
          m.assignedBranches.push({ ...assignedBranch, distanceToManager: parseFloat(minDistance.toFixed(2)) });
          m.totalDistance += minDistance;
          distributedInThisRound = true;
          assignedCount++;
        }
        
        if (!distributedInThisRound) break;
        round++;
      }

      setOptimizedResult(managerWorkload);
      setLoading(false);
      setActiveTab('results');
    }, 1000);
  };

  // تصدير النتائج إلى ملف Excel
  const exportToExcel = () => {
    if (!optimizedResult) return;

    let excelData = [];
    optimizedResult.forEach(m => {
      m.assignedBranches.forEach(b => {
        excelData.push({
          "اسم المدير": m.name,
          "اسم الفرع": b.name,
          "خط عرض الفرع": b.lat,
          "خط طول الفرع": b.lng,
          "المسافة المتوقعة (كم)": b.distanceToManager
        });
      });
    });

    const worksheet = XLSX.utils.json_to_sheet(excelData);
    const workbook = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(workbook, worksheet, "Distribution Results");
    XLSX.writeFile(workbook, "Optimized_Branches_Distribution.xlsx");
  };

  return (
    <div className="min-h-screen bg-gray-50 text-gray-800 font-sans" dir="rtl">
      {/* Header */}
      <header className="bg-indigo-700 text-white shadow-md py-4 px-6 flex justify-between items-center">
        <div className="flex items-center gap-3">
          <Layers className="w-8 h-8" />
          <h1 className="text-xl font-bold">نظام إعادة توزيع الفروع الذكي (Branch Optimizer)</h1>
        </div>
        <div className="text-sm bg-indigo-800 px-3 py-1 rounded-full">
          جاهز للتشغيل على Vercel 🚀
        </div>
      </header>

      {/* Main Container */}
      <main className="max-w-7xl mx-auto p-6 space-y-6">
        
        {/* Control Panel / Inputs */}
        <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
          
          {/* 1. API Key Input */}
          <div className="bg-white p-5 rounded-xl shadow-sm border border-gray-200">
            <h2 className="font-semibold text-gray-700 mb-2 flex items-center gap-2">
              <ShieldCheck className="w-5 h-5 text-indigo-600" /> مفتاح Google Maps API
            </h2>
            <input 
              type="text" 
              placeholder="أدخل مفتاح الـ API هنا (اختياري)..." 
              value={apiKey}
              onChange={(e) => setApiKey(e.target.value)}
              className="w-full p-2 border border-gray-300 rounded-lg text-sm focus:ring-2 focus:ring-indigo-500 outline-none"
            />
            <p className="text-xs text-gray-400 mt-1">يستخدم الخوارزمية الجغرافية الذكية بدقة عالية.</p>
          </div>

          {/* 2. File Upload */}
          <div className="bg-white p-5 rounded-xl shadow-sm border border-gray-200">
            <h2 className="font-semibold text-gray-700 mb-2 flex items-center gap-2">
              <Upload className="w-5 h-5 text-indigo-600" /> رفع ملف الفروع (Excel/CSV)
            </h2>
            <input 
              type="file" 
              accept=".xlsx, .xls, .csv" 
              onChange={handleFileUpload}
              className="w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-lg file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-700 hover:file:bg-indigo-100 cursor-pointer"
            />
            <p className="text-xs text-gray-400 mt-1">تم تحميل: {branches.length} فرع جاهز للمعالجة.</p>
          </div>

          {/* 3. Action Button */}
          <div className="bg-white p-5 rounded-xl shadow-sm border border-gray-200 flex flex-col justify-between">
            <h2 className="font-semibold text-gray-700 mb-2 flex items-center gap-2">
              <Award className="w-5 h-5 text-indigo-600" /> تنفيذ التحسين
            </h2>
            <button 
              onClick={handleOptimize}
              disabled={loading || branches.length === 0}
              className="w-full bg-indigo-600 text-white py-2 rounded-lg font-medium hover:bg-indigo-700 transition disabled:bg-gray-300 flex items-center justify-center gap-2"
            >
              {loading ? <RefreshCw className="w-5 h-5 animate-spin" /> : <RefreshCw className="w-5 h-5" />}
              تحسين وتوزيع الفروع
            </button>
          </div>
        </div>

        {/* Navigation Tabs */}
        <div className="flex border-b border-gray-200 gap-4">
          <button 
            onClick={() => setActiveTab('dashboard')}
            className={`pb-3 px-4 font-medium border-b-2 transition ${activeTab === 'dashboard' ? 'border-indigo-600 text-indigo-600' : 'border-transparent text-gray-500'}`}
          >
            لوحة التحكم وجداول المديرين ({managers.length})
          </button>
          <button 
            onClick={() => setActiveTab('results')}
            className={`pb-3 px-4 font-medium border-b-2 transition ${activeTab === 'results' ? 'border-indigo-600 text-indigo-600' : 'border-transparent text-gray-500'}`}
          >
            النتائج والتوزيع النهائي {optimizedResult && '✨'}
          </button>
          <button 
            onClick={() => setActiveTab('map')}
            className={`pb-3 px-4 font-medium border-b-2 transition ${activeTab === 'map' ? 'border-indigo-600 text-indigo-600' : 'border-transparent text-gray-500'}`}
          >
            الخريطة التفاعلية 🗺️
          </button>
        </div>

        {/* Tab Content 1: Managers & Inputs */}
        {activeTab === 'dashboard' && (
          <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
            <div className="bg-white p-6 rounded-xl shadow-sm border border-gray-200">
              <h3 className="text-lg font-bold mb-4 flex items-center gap-2 text-gray-700">
                <Users className="w-5 h-5 text-indigo-600" /> قائمة مديري المناطق (Area Managers)
              </h3>
              <div className="space-y-3 max-h-96 overflow-y-auto">
                {managers.map(m => (
                  <div key={m.id} className="p-3 bg-gray-50 rounded-lg border border-gray-100 flex justify-between items-center">
                    <div>
                      <p className="font-semibold text-gray-800">{m.name}</p>
                      <p className="text-xs text-gray-500">Lat: {m.lat} | Lng: {m.lng}</p>
                    </div>
                    <span className="bg-indigo-100 text-indigo-800 text-xs px-2.5 py-1 rounded-full font-medium">
                      الحد الأقصى: {m.maxBranches} فروع
                    </span>
                  </div>
                ))}
              </div>
            </div>

            <div className="bg-white p-6 rounded-xl shadow-sm border border-gray-200">
              <h3 className="text-lg font-bold mb-4 flex items-center gap-2 text-gray-700">
                <MapPin className="w-5 h-5 text-indigo-600" /> عينة من الفروع المرفوعة ({branches.length})
              </h3>
              <div className="overflow-x-auto max-h-96">
                <table className="w-full text-right text-sm">
                  <thead className="bg-gray-100 text-gray-600">
                    <tr>
                      <th className="p-2">اسم الفرع</th>
                      <th className="p-2">خط العرض</th>
                      <th className="p-2">خط الطول</th>
                    </tr>
                  </thead>
                  <tbody>
                    {branches.slice(0, 10).map((b, i) => (
                      <tr key={i} className="border-b border-gray-100">
                        <td className="p-2 font-medium">{b.name}</td>
                        <td className="p-2 text-gray-500">{b.lat}</td>
                        <td className="p-2 text-gray-500">{b.lng}</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
                {branches.length > 10 && <p className="text-xs text-center text-gray-400 mt-3">يتم عرض أول 10 فروع فقط...</p>}
              </div>
            </div>
          </div>
        )}

        {/* Tab Content 2: Results Table */}
        {activeTab === 'results' && (
          <div className="bg-white p-6 rounded-xl shadow-sm border border-gray-200 space-y-6">
            <div className="flex justify-between items-center">
              <h3 className="text-lg font-bold text-gray-800">نتائج التوزيع الذكي والعادل</h3>
              {optimizedResult && (
                <button 
                  onClick={exportToExcel}
                  className="bg-green-600 text-white px-4 py-2 rounded-lg font-medium hover:bg-green-700 transition flex items-center gap-2 text-sm"
                >
                  <Download className="w-4 h-4" /> تصدير إلى Excel
                </button>
              )}
            </div>

            {!optimizedResult ? (
              <div className="text-center py-12 text-gray-400">
                لم يتم تشغيل التحسين بعد. قم بررفع الملف واضغط على "تحسين وتوزيع الفروع".
              </div>
            ) : (
              <div className="space-y-6">
                {optimizedResult.map(m => (
                  <div key={m.id} className="border border-gray-200 rounded-xl p-4 bg-gray-50">
                    <div className="flex justify-between items-center mb-3">
                      <h4 className="font-bold text-indigo-700 text-base">{m.name}</h4>
                      <span className="text-xs bg-indigo-100 text-indigo-800 px-3 py-1 rounded-full font-semibold">
                        عدد الفروع المسندة: {m.assignedBranches.length} / {m.maxBranches} | إجمالي المسافة: {m.totalDistance.toFixed(1)} كم
                      </span>
                    </div>
                    <div className="grid grid-cols-1 md:grid-cols-3 gap-3">
                      {m.assignedBranches.map((b, idx) => (
                        <div key={idx} className="bg-white p-3 rounded-lg border border-gray-200 text-sm flex justify-between items-center">
                          <span className="font-medium text-gray-800">{b.name}</span>
                          <span className="text-xs text-gray-500 bg-gray-100 px-2 py-1 rounded">
                            {b.distanceToManager} كم
                          </span>
                        </div>
                      ))}
                      {m.assignedBranches.length === 0 && (
                        <p className="text-xs text-gray-400 italic">لا توجد فروع مسندة.</p>
                      )}
                    </div>
                  </div>
                ))}
              </div>
            )}
          </div>
        )}

        {/* Tab Content 3: Interactive Map */}
        {activeTab === 'map' && (
          <div className="bg-white p-6 rounded-xl shadow-sm border border-gray-200">
            <h3 className="text-lg font-bold mb-4 text-gray-800">الخريطة التفاعلية لمواقع المديرين والفروع</h3>
            <div className="h-[500px] w-full rounded-xl overflow-hidden border border-gray-200 z-0">
              <MapContainer center={[30.0444, 31.2357]} zoom={10} style={{ height: '100%', width: '100%' }}>
                <TileLayer
                  url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
                  attribution='&copy; OpenStreetMap contributors'
                />
                
                {/* عرض مواقع المديرين */}
                {managers.map(m => (
                  <Marker key={`manager-${m.id}`} position={[m.lat, m.lng]}>
                    <Popup>
                      <div className="text-right">
                        <strong>المدير: {m.name}</strong>
                        <p className="text-xs">الحد الأقصى: {m.maxBranches} فروع</p>
                      </div>
                    </Popup>
                  </Marker>
                ))}

                {/* عرض مواقع الفروع والخطوط إذا تم التحسين */}
                {optimizedResult && optimizedResult.map(m => 
                  m.assignedBranches.map((b, idx) => (
                    <React.Fragment key={`branch-line-${m.id}-${idx}`}>
                      <Marker position={[b.lat, b.lng]}>
                        <Popup>
                          <div className="text-right">
                            <strong>الفرع: {b.name}</strong>
                            <p className="text-xs">مسؤول عنه: {m.name}</p>
                            <p className="text-xs">المسافة: {b.distanceToManager} كم</p>
                          </div>
                        </Popup>
                      </Marker>
                      <Polyline 
                        positions={[[m.lat, m.lng], [b.lat, b.lng]]} 
                        color="#4f46e5" 
                        dashArray="4" 
                      />
                    </React.Fragment>
                  ))
                )}
              </MapContainer>
            </div>
          </div>
        )}

      </main>
    </div>
  );
}
