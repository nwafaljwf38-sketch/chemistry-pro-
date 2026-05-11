import { useState, useEffect, useRef } from "react";

const C = {
  bg: "#060b14", card: "#0d1520", cardHigh: "#111e2e",
  border: "#1a2d42", borderHigh: "#1e3a54",
  accent: "#00e5ff", accent2: "#7c3aed",
  green: "#00d68f", red: "#ff4d6d", yellow: "#ffd60a",
  orange: "#ff6b35", text: "#e8f4f8", muted: "#4a6b82", dim: "#8aafc4",
};

const TOPICS = [
  { id: "alkyl",     label: "مجموعات الألكيل",         icon: "⛓️", color: "#7c3aed" },
  { id: "halides",   label: "هاليدات الألكيل",          icon: "🧂", color: "#0ea5e9" },
  { id: "aryl",      label: "مجموعات الأريل",           icon: "🔷", color: "#06b6d4" },
  { id: "alcohols",  label: "الكحولات",                 icon: "🍶", color: "#10b981" },
  { id: "ethers",    label: "الإيثرات",                 icon: "💧", color: "#3b82f6" },
  { id: "aldehydes", label: "الألدهيدات والكيتونات",    icon: "⚗️", color: "#f59e0b" },
  { id: "acids",     label: "الأحماض الكربوكسيلية",     icon: "🔴", color: "#ef4444" },
  { id: "esters",    label: "الإسترات",                 icon: "🌸", color: "#ec4899" },
  { id: "amines",    label: "الأمينات",                 icon: "🟣", color: "#8b5cf6" },
  { id: "naming",    label: "تسمية المركبات",           icon: "🏷️", color: "#14b8a6" },
  { id: "isomers",   label: "المتشكلات",                icon: "🔀", color: "#f97316" },
  { id: "reactions", label: "التفاعلات العضوية",        icon: "⚡", color: "#eab308" },
  { id: "polymers",  label: "البوليمرات",               icon: "🧵", color: "#84cc16" },
  { id: "benzene",   label: "البنزين ومشتقاته",         icon: "⬡",  color: "#06b6d4" },
  { id: "mixed",     label: "مزيج شامل 🎲",             icon: "🎯", color: "#00e5ff" },
];

const TOPIC_PROMPTS = {
  alkyl:     "مجموعات الألكيل (ميثيل، إيثيل، بروبيل، بيوتيل...) وتسميتها وخواصها وارتباطها بالمركبات العضوية",
  halides:   "هاليدات الألكيل: التسمية، التصنيف (أولي/ثانوي/ثالثي)، تفاعلات الإزالة والإحلال، أمثلة: كلوروميثان، بروموإيثان، يودوبروبان",
  aryl:      "مجموعات الأريل المشتقة من البنزين (فينيل، بنزيل)، خواصها، تسميتها، ارتباطها بالمركبات العضوية الأخرى",
  alcohols:  "الكحولات: تسمية IUPAC، تصنيف (أولي/ثانوي/ثالثي)، خواص فيزيائية، تفاعلات الأكسدة والاستخلاص والتكسير",
  ethers:    "الإيثرات: بناؤها، تسميتها، مقارنتها بالكحولات في درجة الغليان، طرق تحضيرها، خواصها الكيميائية",
  aldehydes: "الألدهيدات والكيتونات: تسميتها، الفرق بينهما، اختبار تولنز وفيلينج، تفاعل الإضافة النووية، أمثلة عملية",
  acids:     "الأحماض الكربوكسيلية: التسمية، الخواص الحمضية، التفاعلات (أسترة، تحييد، تفاعل مع PCl5)، أمثلة: حمض الخليك، الفورميك",
  esters:    "الإسترات: تكوينها من حمض وكحول، التسمية، التحلل المائي، الصابنة، الاستخدامات العملية في الروائح",
  amines:    "الأمينات: تصنيفها (أولية/ثانوية/ثالثية)، تسميتها، خواصها القاعدية، تفاعلاتها مع الأحماض، الأنيلين",
  naming:    "قواعد تسمية IUPAC للمركبات العضوية: اختيار السلسلة الرئيسية، الترقيم، ترتيب المجموعات، اللواحق والسوابق",
  isomers:   "المتشكلات: بنيوية وفراغية (cis/trans)، أنواع المتشكلات البنيوية (سلسلة، موضع، وظيفي)، المتشكلات الضوئية",
  reactions: "التفاعلات العضوية: الإضافة، الاستبدال، الحذف، البلمرة، التأكسد والاختزال، التحلل المائي وأمثلة لكل نوع",
  polymers:  "البوليمرات: بلمرة الإضافة والتكثيف، المونومر والبوليمر، البوليمرات الطبيعية (نشا، بروتين) والصناعية (بولي إيثيلين، نايلون)",
  benzene:   "البنزين: بنيته، خواصه الفيزيائية والكيميائية، تفاعلات الإحلال الكهربي (هلجنة، نترة، سلفنة، ألكلة)، مشتقاته",
  mixed:     "مزيج شامل من جميع موضوعات الكيمياء العضوية لصف ثاني ثانوي في المملكة العربية السعودية",
};

const INITIAL_LB = [
  { name: "سارة العتيبي",     score: 47, topic: "الكحولات" },
  { name: "محمد الغامدي",     score: 39, topic: "هاليدات الألكيل" },
  { name: "نورة القحطاني",    score: 35, topic: "مزيج شامل" },
  { name: "عبدالله الزهراني", score: 28, topic: "البنزين" },
  { name: "ريم الدوسري",      score: 21, topic: "الأحماض" },
];

// Animated molecule background
function MoleculeBg() {
  const items = useRef(
    Array.from({ length: 16 }, (_, i) => ({
      id: i, x: Math.random() * 100, y: Math.random() * 100,
      r: Math.random() * 16 + 7, dur: Math.random() * 16 + 10, del: Math.random() * 7,
      col: [C.accent, C.accent2, C.green, C.yellow, C.orange][i % 5],
      sym: ["C","H","O","N","Cl","Br","OH","CH₃","NH₂","CO","⬡","F"][i % 12],
    }))
  ).current;
  const lines = useRef(
    Array.from({ length: 12 }, (_, i) => ({
      id: i, x1: Math.random() * 100, y1: Math.random() * 100,
      x2: Math.random() * 100, y2: Math.random() * 100,
      dur: Math.random() * 18 + 10, del: Math.random() * 5,
    }))
  ).current;
  return (
    <svg style={{ position:"fixed",inset:0,width:"100%",height:"100%",pointerEvents:"none",zIndex:0,opacity:0.11 }}>
      <defs>
        <filter id="glow2"><feGaussianBlur stdDeviation="2.5" result="b"/><feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge></filter>
      </defs>
      {lines.map(l => (
        <line key={l.id} x1={`${l.x1}%`} y1={`${l.y1}%`} x2={`${l.x2}%`} y2={`${l.y2}%`}
          stroke={C.accent} strokeWidth="0.8" filter="url(#glow2)" opacity="0.5">
          <animate attributeName="opacity" values="0.2;0.7;0.2" dur={`${l.dur}s`} begin={`${l.del}s`} repeatCount="indefinite"/>
        </line>
      ))}
      {items.map(a => (
        <g key={a.id}>
          <circle cx={`${a.x}%`} cy={`${a.y}%`} r={a.r} fill={a.col} filter="url(#glow2)">
            <animate attributeName="opacity" values="0.3;0.9;0.3" dur={`${a.dur}s`} begin={`${a.del}s`} repeatCount="indefinite"/>
            <animateTransform attributeName="transform" type="translate" values="0,0;5,9;-3,5;0,0" dur={`${a.dur}s`} begin={`${a.del}s`} repeatCount="indefinite"/>
          </circle>
          <text x={`${a.x}%`} y={`${a.y}%`} textAnchor="middle" dominantBaseline="middle"
            fontSize={a.r * 0.65} fill="#fff" fontFamily="monospace" fontWeight="bold">{a.sym}</text>
        </g>
      ))}
    </svg>
  );
}

export default function App() {
  const [screen, setScreen]           = useState("home");
  const [nameInput, setNameInput]     = useState("");
  const [playerName, setPlayerName]   = useState("");
  const [topic, setTopic]             = useState(null);
  const [score, setScore]             = useState(0);
  const [qNum, setQNum]               = useState(0);
  const TOTAL                         = 5;
  const [question, setQuestion]       = useState(null);
  const [loading, setLoading]         = useState(false);
  const [chosen, setChosen]           = useState(null);
  const [revealed, setRevealed]       = useState(false);
  const [lb, setLb]                   = useState(INITIAL_LB);
  const [timeLeft, setTimeLeft]       = useState(30);
  const [timerOn, setTimerOn]         = useState(false);
  const [streak, setStreak]           = useState(0);
  const [maxStreak, setMaxStreak]     = useState(0);
  const timerRef = useRef(null);

  useEffect(() => {
    if (timerOn && timeLeft > 0) {
      timerRef.current = setTimeout(() => setTimeLeft(t => t - 1), 1000);
    } else if (timerOn && timeLeft === 0 && !revealed) {
      setTimerOn(false); setRevealed(true); setStreak(0);
    }
    return () => clearTimeout(timerRef.current);
  }, [timerOn, timeLeft, revealed]);

  const fetchQ = async () => {
    setLoading(true); setChosen(null); setRevealed(false); setTimeLeft(30);
    const desc = TOPIC_PROMPTS[topic?.id] || TOPIC_PROMPTS.mixed;
    const sys = `أنت أستاذ كيمياء عضوية للصف الثاني الثانوي في المملكة العربية السعودية.
الموضوع: ${desc}
أسلوب الأسئلة: متنوع – مفاهيمي، تسمية IUPAC، تفاعلات، أمثلة، خواص.
المستوى: متوسط إلى صعب مناسب لثاني ثانوي.
أجب فقط بـ JSON هكذا بدون أي نص خارجه:
{"question":"نص السؤال","options":["أ","ب","ج","د"],"correct":0,"explanation":"شرح مختصر"}`;
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514", max_tokens: 1000,
          system: sys,
          messages: [{ role: "user", content: `سؤال رقم ${qNum + 1}` }],
        }),
      });
      const data = await res.json();
      const txt = data.content.map(c => c.text || "").join("");
      setQuestion(JSON.parse(txt.replace(/```json|```/g, "").trim()));
      setTimerOn(true);
    } catch {
      setQuestion({ question:"ما الصيغة البنيوية لمجموعة الميثيل؟", options:["–CH₃","–C₂H₅","–C₃H₇","–CH₂–"], correct:0, explanation:"مجموعة الميثيل هي –CH₃ وهي أبسط مجموعة ألكيل." });
      setTimerOn(true);
    }
    setLoading(false);
  };

  useEffect(() => { if (screen === "quiz") fetchQ(); }, [screen, qNum]);

  const handleAnswer = (idx) => {
    if (chosen !== null || revealed) return;
    clearTimeout(timerRef.current); setTimerOn(false);
    setChosen(idx); setRevealed(true);
    if (idx === question.correct) {
      const pts = timeLeft > 20 ? 3 : timeLeft > 10 ? 2 : 1;
      setScore(s => s + pts + (streak >= 2 ? 1 : 0));
      const ns = streak + 1; setStreak(ns); setMaxStreak(m => Math.max(m, ns));
    } else setStreak(0);
  };

  const nextQ = () => {
    if (qNum + 1 >= TOTAL) {
      setLb(prev => [...prev, { name: playerName, score, topic: topic?.label }].sort((a,b)=>b.score-a.score).slice(0,10));
      setScreen("result");
    } else setQNum(n => n + 1);
  };

  const startQuiz = () => {
    if (!nameInput.trim() || !topic) return;
    setPlayerName(nameInput.trim()); setScore(0); setQNum(0); setStreak(0); setMaxStreak(0);
    setScreen("quiz");
  };

  const tCol = timeLeft > 20 ? C.green : timeLeft > 10 ? C.yellow : C.red;
  const labels = ["أ","ب","ج","د"];

  // ── HOME ──────────────────────────────────────────────────────────────────
  if (screen === "home") return (
    <div style={A.app}>
      <style>{CSS}</style>
      <MoleculeBg />
      <div style={A.page}>
        <div style={A.hero}>
          <div style={{ fontSize: 68 }}>⚗️</div>
          <h1 style={A.title}>كيمياء برو</h1>
          <p style={A.sub}>ثاني ثانوي • كيمياء عضوية • تنافس مع زملاءك</p>
          <div style={A.chips}>
            {["⏱️ 30 ثانية لكل سؤال","🔥 نقاط للسرعة","🏆 لوحة متصدرين"].map(t=>(
              <span key={t} style={A.chip}>{t}</span>
            ))}
          </div>
        </div>

        <div style={A.sec}>
          <div style={A.lbl}>اسمك</div>
          <input style={A.inp} placeholder="اكتب اسمك..." value={nameInput}
            onChange={e=>setNameInput(e.target.value)}
            onKeyDown={e=>e.key==="Enter"&&startQuiz()} />
        </div>

        <div style={A.sec}>
          <div style={A.lbl}>اختر الموضوع</div>
          <div style={A.grid}>
            {TOPICS.map(t=>(
              <button key={t.id} style={{ ...A.tBtn, ...(topic?.id===t.id?{borderColor:t.color,background:`${t.color}1a`,color:t.color}:{}) }}
                onClick={()=>setTopic(t)}>
                {topic?.id===t.id && <span style={{ position:"absolute",top:5,left:7,fontSize:10,color:t.color }}>✔</span>}
                <span style={{ fontSize:22 }}>{t.icon}</span>
                <span style={{ fontSize:11.5,lineHeight:1.3,textAlign:"center" }}>{t.label}</span>
              </button>
            ))}
          </div>
        </div>

        <button style={{ ...A.btn, opacity:(!nameInput.trim()||!topic)?0.4:1 }} onClick={startQuiz} disabled={!nameInput.trim()||!topic}>
          🚀 ابدأ التحدي
        </button>
        <button style={A.ghost} onClick={()=>setScreen("leaderboard")}>🏆 لوحة المتصدرين</button>
      </div>
    </div>
  );

  // ── QUIZ ──────────────────────────────────────────────────────────────────
  if (screen === "quiz") return (
    <div style={A.app}>
      <style>{CSS}</style>
      <MoleculeBg />
      <div style={A.page}>
        <div style={A.topBar}>
          <div style={A.sPill}>⚡ {score} نقطة</div>
          {streak>=2 && <div style={A.firePill}>🔥 {streak}x</div>}
          <div style={{ flex:1 }}/>
          <div style={A.qCnt}>{qNum+1} / {TOTAL}</div>
        </div>

        <div style={{ ...A.tTag, borderColor:topic?.color, color:topic?.color }}>
          {topic?.icon} {topic?.label}
        </div>

        <div style={A.timerRow}>
          <span style={{ color:tCol,fontWeight:800,fontSize:20,minWidth:28 }}>{timeLeft}</span>
          <div style={A.timerTrack}>
            <div style={{ ...A.timerFill, width:`${(timeLeft/30)*100}%`, background:tCol }} />
          </div>
        </div>

        <div style={A.qCard}>
          {loading ? (
            <div style={A.loadBox}>
              <div style={{ fontSize:44,animation:"spin 2s linear infinite" }}>⚗️</div>
              <p style={{ color:C.muted,marginTop:12 }}>جاري توليد السؤال بالذكاء الاصطناعي...</p>
            </div>
          ) : question ? (
            <>
              <p style={A.qTxt}>{question.question}</p>
              <div style={{ display:"flex",flexDirection:"column",gap:10 }}>
                {question.options.map((opt,i)=>{
                  let bc=C.border, bg=C.card, col=C.text;
                  if (revealed) {
                    if (i===question.correct) { bc=C.green; bg=`${C.green}18`; col=C.green; }
                    else if (i===chosen)      { bc=C.red;   bg=`${C.red}18`;   col=C.red; }
                  } else if (chosen===i) { bc=C.accent; bg=`${C.accent}10`; }
                  return (
                    <button key={i} onClick={()=>handleAnswer(i)}
                      style={{ ...A.optBtn, borderColor:bc, background:bg, color:col }}>
                      <span style={{ ...A.optLbl, background:bc===C.border?C.border:`${bc}28`, color:bc===C.border?C.dim:bc }}>
                        {labels[i]}
                      </span>
                      <span style={{ flex:1,textAlign:"right" }}>{opt}</span>
                      {revealed && i===question.correct && <span>✅</span>}
                      {revealed && i===chosen && i!==question.correct && <span>❌</span>}
                    </button>
                  );
                })}
              </div>
              {revealed && (
                <div style={{ marginTop:16 }}>
                  <div style={A.explain}>💡 {question.explanation}</div>
                  {chosen===question.correct
                    ? <div style={{ textAlign:"center",color:C.green,fontWeight:800,fontSize:15,marginTop:10 }}>
                        +{timeLeft>20?3:timeLeft>10?2:1}{streak>=2?" +1 🔥":""} نقطة!
                      </div>
                    : <div style={{ textAlign:"center",color:C.red,fontWeight:700,fontSize:13,marginTop:10 }}>
                        الإجابة الصحيحة: {question.options[question.correct]}
                      </div>
                  }
                  <button style={{ ...A.btn, marginTop:14 }} onClick={nextQ}>
                    {qNum+1>=TOTAL ? "🏁 عرض النتيجة" : "السؤال التالي ←"}
                  </button>
                </div>
              )}
            </>
          ) : null}
        </div>
      </div>
    </div>
  );

  // ── RESULT ────────────────────────────────────────────────────────────────
  if (screen === "result") {
    const pct = Math.round((score/(TOTAL*3))*100);
    const [em,msg] = pct>=80?["🏆","ممتاز! أنت نجم الكيمياء"]:pct>=60?["🥈","أداء رائع! استمر"]:pct>=40?["🥉","جيد، تدرّب أكثر"]:["📚","راجع الموضوع وأعد المحاولة"];
    const rank = lb.findIndex(e=>e.name===playerName)+1;
    return (
      <div style={A.app}>
        <style>{CSS}</style>
        <MoleculeBg />
        <div style={A.page}>
          <div style={{ textAlign:"center",paddingBlock:20 }}>
            <div style={{ fontSize:80,animation:"pop .5s ease" }}>{em}</div>
            <h2 style={{ ...A.title,marginTop:8 }}>{score} نقطة</h2>
            <p style={{ color:C.dim,fontSize:16,marginTop:6 }}>{msg}</p>
          </div>
          <div style={{ display:"grid",gridTemplateColumns:"1fr 1fr",gap:10 }}>
            {[["الأسئلة",`${TOTAL}`],["النسبة",`${pct}%`],["ترتيبك",rank>0?`#${rank}`:"—"],["أعلى سلسلة",`${maxStreak} 🔥`]].map(([l,v])=>(
              <div key={l} style={A.statCard}>
                <div style={{ fontSize:24,fontWeight:900,color:C.accent }}>{v}</div>
                <div style={{ fontSize:12,color:C.muted,marginTop:4 }}>{l}</div>
              </div>
            ))}
          </div>
          <button style={A.btn} onClick={()=>{ setScore(0);setQNum(0);setStreak(0);setMaxStreak(0);setScreen("quiz"); }}>🔄 تحدٍّ جديد – نفس الموضوع</button>
          <button style={A.ghost} onClick={()=>setScreen("home")}>🔁 اختر موضوعاً آخر</button>
          <button style={{ ...A.ghost, borderColor:C.yellow, color:C.yellow }} onClick={()=>setScreen("leaderboard")}>🏆 لوحة المتصدرين</button>
        </div>
      </div>
    );
  }

  // ── LEADERBOARD ───────────────────────────────────────────────────────────
  if (screen === "leaderboard") return (
    <div style={A.app}>
      <style>{CSS}</style>
      <MoleculeBg />
      <div style={A.page}>
        <div style={A.hero}>
          <div style={{ fontSize:56 }}>🏆</div>
          <h1 style={A.title}>المتصدرون</h1>
          <p style={A.sub}>أفضل طلاب كيمياء برو</p>
        </div>
        {lb.map((e,i)=>(
          <div key={i} style={{ ...A.lRow, ...(i===0?{borderColor:C.yellow,background:`${C.yellow}0d`}:{}) }}>
            <div style={{ ...A.rankBadge, background:i===0?C.yellow:i===1?"#94a3b8":i===2?"#cd7c2f":C.border }}>
              {i<3?["🥇","🥈","🥉"][i]:i+1}
            </div>
            <div style={{ flex:1 }}>
              <div style={{ fontWeight:700,fontSize:15 }}>{e.name}</div>
              <div style={{ fontSize:11,color:C.muted,marginTop:2 }}>{e.topic}</div>
            </div>
            <div style={{ fontWeight:800,fontSize:22,color:i===0?C.yellow:C.text }}>
              {e.score}<span style={{ fontSize:11,color:C.muted,fontWeight:400 }}> نقطة</span>
            </div>
          </div>
        ))}
        <button style={{ ...A.ghost,marginTop:12 }} onClick={()=>setScreen("home")}>← رجوع</button>
      </div>
    </div>
  );
}

const A = {
  app:  { minHeight:"100vh", background:C.bg, color:C.text, fontFamily:"'Cairo',sans-serif", direction:"rtl", position:"relative" },
  page: { position:"relative",zIndex:1,maxWidth:560,margin:"0 auto",padding:"24px 16px 56px",display:"flex",flexDirection:"column",gap:14 },
  hero: { textAlign:"center",paddingBlock:12 },
  title:{ fontSize:34,fontWeight:900,margin:0,background:`linear-gradient(135deg,${C.accent},${C.accent2})`,WebkitBackgroundClip:"text",WebkitTextFillColor:"transparent" },
  sub:  { color:C.muted,fontSize:14,marginTop:6 },
  chips:{ display:"flex",gap:8,justifyContent:"center",flexWrap:"wrap",marginTop:14 },
  chip: { background:"#0d1a29",border:`1px solid ${C.border}`,borderRadius:99,padding:"4px 12px",fontSize:12,color:C.dim },
  sec:  { display:"flex",flexDirection:"column",gap:8 },
  lbl:  { fontSize:13,color:C.dim,fontWeight:700 },
  inp:  { background:"#0b1624",border:`1.5px solid ${C.border}`,borderRadius:12,padding:"13px 14px",color:C.text,fontSize:15,outline:"none",textAlign:"right",fontFamily:"'Cairo',sans-serif",transition:"border-color .2s" },
  grid: { display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:8 },
  tBtn: { position:"relative",background:"#0b1624",border:`1.5px solid ${C.border}`,borderRadius:12,padding:"12px 8px",display:"flex",flexDirection:"column",alignItems:"center",gap:6,color:C.muted,cursor:"pointer",transition:"all .18s",fontFamily:"'Cairo',sans-serif",fontSize:13,minHeight:80 },
  btn:  { width:"100%",padding:"14px",borderRadius:12,border:"none",background:`linear-gradient(135deg,${C.accent},${C.accent2})`,color:"#fff",fontSize:16,fontWeight:800,cursor:"pointer",fontFamily:"'Cairo',sans-serif",transition:"all .2s" },
  ghost:{ width:"100%",padding:"13px",borderRadius:12,background:"transparent",border:`1.5px solid ${C.border}`,color:C.dim,fontSize:15,fontWeight:700,cursor:"pointer",fontFamily:"'Cairo',sans-serif",transition:"all .2s" },
  topBar:{ display:"flex",alignItems:"center",gap:8 },
  sPill:{ background:`${C.accent}18`,border:`1px solid ${C.accent}`,borderRadius:99,padding:"5px 14px",fontSize:13,color:C.accent,fontWeight:700 },
  firePill:{ background:`${C.yellow}18`,border:`1px solid ${C.yellow}`,borderRadius:99,padding:"5px 12px",fontSize:13,color:C.yellow,fontWeight:700 },
  qCnt: { color:C.muted,fontSize:14,fontWeight:700 },
  tTag: { display:"inline-flex",alignSelf:"flex-start",alignItems:"center",gap:6,border:"1px solid",borderRadius:99,padding:"4px 12px",fontSize:12,fontWeight:700 },
  timerRow:{ display:"flex",alignItems:"center",gap:10 },
  timerTrack:{ flex:1,height:6,background:C.border,borderRadius:99,overflow:"hidden" },
  timerFill:{ height:"100%",borderRadius:99,transition:"width 1s linear, background .5s" },
  qCard:{ background:C.card,border:`1px solid ${C.border}`,borderRadius:18,padding:"24px 20px" },
  loadBox:{ textAlign:"center",padding:"32px 0" },
  qTxt: { fontSize:18,fontWeight:700,lineHeight:1.8,textAlign:"center",marginBottom:20 },
  optBtn:{ display:"flex",alignItems:"center",gap:12,padding:"13px 14px",borderRadius:12,border:"1.5px solid",background:C.card,color:C.text,cursor:"pointer",fontFamily:"'Cairo',sans-serif",fontSize:14,fontWeight:600,textAlign:"right",transition:"all .18s",width:"100%" },
  optLbl:{ borderRadius:7,width:28,height:28,display:"flex",alignItems:"center",justifyContent:"center",fontSize:12,fontWeight:900,flexShrink:0 },
  explain:{ background:`${C.green}12`,border:`1px solid ${C.green}30`,borderRadius:12,padding:"12px 14px",fontSize:14,color:"#6ee7b7",lineHeight:1.7 },
  statCard:{ background:"#0b1624",border:`1px solid ${C.border}`,borderRadius:14,padding:16,textAlign:"center" },
  lRow: { display:"flex",alignItems:"center",gap:12,padding:"13px 14px",background:"#0b1624",border:`1px solid ${C.border}`,borderRadius:14 },
  rankBadge:{ width:34,height:34,borderRadius:"50%",display:"flex",alignItems:"center",justifyContent:"center",fontSize:15,fontWeight:900,color:"#fff",flexShrink:0 },
};

const CSS = `
  @import url('https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700;800;900&display=swap');
  *{box-sizing:border-box;margin:0;padding:0;}
  body{background:#060b14;}
  @keyframes spin{to{transform:rotate(360deg);}}
  @keyframes pop{from{transform:scale(.4);opacity:0;}to{transform:scale(1);opacity:1;}}
  input:focus{border-color:#00e5ff!important;box-shadow:0 0 0 3px #00e5ff18;}
  button:not(:disabled):hover{opacity:.87;transform:translateY(-1px);}
  button:disabled{cursor:not-allowed;}
  ::-webkit-scrollbar{width:4px;}
  ::-webkit-scrollbar-thumb{background:#1a2d42;border-radius:99px;}
`;
