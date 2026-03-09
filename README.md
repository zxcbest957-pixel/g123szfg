import { useState, useEffect } from "react";

const QUOTES = [
  "Ти вже робиш неможливе — просто живеш 💛",
  "Маленькі кроки теж рухають гори 🌸",
  "Сьогодні — ідеальний день почати 🌷",
  "Ти дивовижна мама і людина ✨",
  "Відпочинок — теж продуктивність 🌿",
];

const CAT_COLORS = ["#F9C6D0","#FAD4C0","#D4EAD0","#C8E6F0","#F5E6C8","#E8D4F0","#F0E8C8","#D4F0E8"];
const CAT_ACCENTS = ["#E8899A","#E89A72","#7EBA78","#6BAED6","#D4A853","#B07EC8","#C4A040","#40B090"];
const CAT_ICONS = ["📚","🎨","🧘‍♀️","🌱","🍳","💄","🎵","🏃‍♀️","💻","🧹","🛒","✂️","🌺","⭐"];

export default function SoftBrightPlanner() {
  const [screen, setScreen] = useState("home");
  const [tasks, setTasks] = useState([]);
  const [notes, setNotes] = useState([]);
  const [categories, setCategories] = useState([]);
  const [loaded, setLoaded] = useState(false);
  const [activeTab, setActiveTab] = useState("today");

  const [showAddTask, setShowAddTask] = useState(false);
  const [newTaskText, setNewTaskText] = useState("");
  const [newTaskCat, setNewTaskCat] = useState(null);
  const [newTaskPriority, setNewTaskPriority] = useState(false);
  const [celebration, setCelebration] = useState(null);

  const [showAddCat, setShowAddCat] = useState(false);
  const [newCatName, setNewCatName] = useState("");
  const [newCatIcon, setNewCatIcon] = useState("⭐");
  const [newCatColorIdx, setNewCatColorIdx] = useState(0);

  const [newNote, setNewNote] = useState("");

  const [quote] = useState(QUOTES[Math.floor(Math.random() * QUOTES.length)]);
  const today = new Date();
  const dateStr = today.toLocaleDateString("uk-UA", { weekday: "long", day: "numeric", month: "long" });

  useEffect(() => {
    async function load() {
      try {
        const [t, n, c] = await Promise.all([
          window.storage.get("sb_tasks").catch(() => null),
          window.storage.get("sb_notes").catch(() => null),
          window.storage.get("sb_cats").catch(() => null),
        ]);
        if (t) setTasks(JSON.parse(t.value));
        if (n) setNotes(JSON.parse(n.value));
        if (c) setCategories(JSON.parse(c.value));
      } catch (e) {}
      setLoaded(true);
    }
    load();
  }, []);

  useEffect(() => { if (loaded) window.storage.set("sb_tasks", JSON.stringify(tasks)).catch(() => {}); }, [tasks, loaded]);
  useEffect(() => { if (loaded) window.storage.set("sb_notes", JSON.stringify(notes)).catch(() => {}); }, [notes, loaded]);
  useEffect(() => { if (loaded) window.storage.set("sb_cats", JSON.stringify(categories)).catch(() => {}); }, [categories, loaded]);

  const todayTasks = tasks.filter(t => t.day === "today");
  const tomorrowTasks = tasks.filter(t => t.day === "tomorrow");
  const currentTasks = activeTab === "today" ? todayTasks : tomorrowTasks;
  const doneTodayCount = todayTasks.filter(t => t.done).length;

  function completeTask(id) {
    const task = tasks.find(t => t.id === id);
    if (!task?.done) { setCelebration(id); setTimeout(() => setCelebration(null), 1000); }
    setTasks(prev => prev.map(t => t.id === id ? { ...t, done: !t.done } : t));
  }

  function deleteTask(id) { setTasks(prev => prev.filter(t => t.id !== id)); }

  function addTask() {
    if (!newTaskText.trim() || !newTaskCat) return;
    setTasks(prev => [...prev, {
      id: Date.now(), catId: newTaskCat, text: newTaskText.trim(),
      priority: newTaskPriority, done: false, day: activeTab
    }]);
    setNewTaskText(""); setNewTaskPriority(false); setShowAddTask(false);
  }

  function moveTask(id, day) { setTasks(prev => prev.map(t => t.id === id ? { ...t, day } : t)); }

  function addNote() {
    if (!newNote.trim()) return;
    const now = new Date();
    const time = now.getHours().toString().padStart(2, "0") + ":" + now.getMinutes().toString().padStart(2, "0");
    setNotes(prev => [{ id: Date.now(), text: newNote.trim(), time }, ...prev]);
    setNewNote("");
  }

  function deleteNote(id) { setNotes(prev => prev.filter(n => n.id !== id)); }

  function addCategory() {
    if (!newCatName.trim()) return;
    setCategories(prev => [...prev, {
      id: Date.now(), name: newCatName.trim(),
      icon: newCatIcon, color: CAT_COLORS[newCatColorIdx], accent: CAT_ACCENTS[newCatColorIdx]
    }]);
    setNewCatName(""); setNewCatIcon("⭐"); setNewCatColorIdx(0); setShowAddCat(false);
  }

  function deleteCategory(id) {
    setCategories(prev => prev.filter(c => c.id !== id));
    setTasks(prev => prev.filter(t => t.catId !== id));
  }

  const tasksByCategory = categories.map(cat => ({
    cat, tasks: currentTasks.filter(t => t.catId === cat.id)
  })).filter(g => g.tasks.length > 0);

  if (!loaded) return (
    <div style={{ display: "flex", alignItems: "center", justifyContent: "center", height: "100vh",
      background: "linear-gradient(160deg,#FFF5F8,#FFF9F5)", fontFamily: "Nunito,sans-serif" }}>
      <div style={{ fontSize: 52 }}>🌸</div>
    </div>
  );

  const S = {
    app: { fontFamily: "'Nunito',sans-serif", background: "linear-gradient(160deg,#FFF5F8 0%,#FFF9F5 50%,#F8F5FF 100%)",
      minHeight: "100vh", maxWidth: 420, margin: "0 auto", position: "relative" },
    header: { background: "linear-gradient(135deg,#FFB5C8 0%,#FFCBA4 100%)", padding: "20px 20px 24px",
      borderRadius: "0 0 32px 32px", boxShadow: "0 8px 32px rgba(255,150,180,0.25)", position: "relative", overflow: "hidden" },
    d1: { position: "absolute", top: -20, right: -20, width: 110, height: 110, background: "rgba(255,255,255,0.18)", borderRadius: "50%" },
    d2: { position: "absolute", bottom: -30, left: -15, width: 75, height: 75, background: "rgba(255,255,255,0.13)", borderRadius: "50%" },
    dateText: { color: "rgba(255,255,255,0.85)", fontSize: 12, fontWeight: 700, textTransform: "capitalize" },
    title: { color: "#fff", fontSize: 24, fontWeight: 900, margin: "2px 0 10px", textShadow: "0 2px 12px rgba(200,80,100,0.2)" },
    quoteBox: { background: "rgba(255,255,255,0.32)", backdropFilter: "blur(8px)", borderRadius: 14,
      padding: "9px 13px", color: "rgba(120,40,60,0.85)", fontSize: 12, fontWeight: 600, fontStyle: "italic",
      border: "1px solid rgba(255,255,255,0.45)" },
    progRow: { display: "flex", alignItems: "center", gap: 10, marginTop: 12 },
    progBar: { flex: 1, height: 7, background: "rgba(255,255,255,0.35)", borderRadius: 99, overflow: "hidden" },
    progFill: p => ({ height: "100%", width: `${p}%`, background: "#fff", borderRadius: 99, transition: "width 0.5s ease", boxShadow: "0 0 8px rgba(255,255,255,0.8)" }),
    progTxt: { color: "rgba(255,255,255,0.9)", fontSize: 12, fontWeight: 700, whiteSpace: "nowrap" },
    tabs: { display: "flex", gap: 8, padding: "14px 16px 0" },
    tab: a => ({ flex: 1, padding: "10px 0", borderRadius: 16, border: "none", cursor: "pointer",
      fontFamily: "inherit", fontWeight: 700, fontSize: 14, transition: "all 0.25s",
      background: a ? "linear-gradient(135deg,#FFB5C8,#FFCBA4)" : "rgba(255,255,255,0.7)",
      color: a ? "#fff" : "#C08090", boxShadow: a ? "0 4px 16px rgba(255,150,180,0.3)" : "none" }),
    body: { padding: "14px 16px 100px" },
    secTit: { fontWeight: 800, fontSize: 18, color: "#C04060", marginBottom: 14, marginTop: 4 },
    empty: { textAlign: "center", padding: "40px 20px", color: "#D0A0B0" },
    catBadge: cat => ({ background: cat.color, borderRadius: 12, padding: "6px 13px",
      display: "inline-flex", alignItems: "center", gap: 6, marginBottom: 10, boxShadow: `0 2px 12px ${cat.color}80` }),
    catBadgeName: cat => ({ color: cat.accent, fontWeight: 800, fontSize: 13 }),
    taskCard: (task, cat) => ({
      background: task.done ? "rgba(255,255,255,0.45)" : "rgba(255,255,255,0.92)",
      borderRadius: 17, padding: "11px 13px", marginBottom: 7,
      display: "flex", alignItems: "center", gap: 9,
      boxShadow: task.done ? "none" : "0 2px 14px rgba(200,100,120,0.09)",
      border: `1.5px solid ${task.done ? cat.color + "55" : cat.color}`,
      opacity: task.done ? 0.6 : 1,
      transition: "all 0.35s cubic-bezier(0.34,1.56,0.64,1)",
      transform: celebration === task.id ? "scale(1.04)" : "scale(1)",
    }),
    taskTxt: done => ({ flex: 1, fontSize: 14, fontWeight: done ? 500 : 600,
      color: done ? "#C0A0A8" : "#5A3040", textDecoration: done ? "line-through" : "none", lineHeight: 1.4 }),
    prioDot: { width: 7, height: 7, background: "linear-gradient(135deg,#FF8FAB,#FFB347)",
      borderRadius: "50%", boxShadow: "0 0 6px rgba(255,150,100,0.55)", flexShrink: 0 },
    doneBtn: (done, cat) => ({ width: 34, height: 34, borderRadius: "50%", border: "none", cursor: "pointer",
      background: done ? `linear-gradient(135deg,${cat.accent},${cat.color})` : `linear-gradient(135deg,${cat.color},#FFF0F5)`,
      display: "flex", alignItems: "center", justifyContent: "center", fontSize: done ? 15 : 13, flexShrink: 0,
      boxShadow: done ? `0 4px 12px ${cat.accent}55` : "none",
      transition: "all 0.3s cubic-bezier(0.34,1.56,0.64,1)", transform: done ? "scale(1.1)" : "scale(1)" }),
    iconBtn: { background: "rgba(200,150,170,0.12)", border: "none", borderRadius: 8,
      padding: "3px 7px", cursor: "pointer", fontSize: 11, color: "#C08090", fontFamily: "inherit", fontWeight: 700, flexShrink: 0 },
    addBtn: { width: "100%", padding: "13px", borderRadius: 18, border: "2px dashed #F9C6D0",
      background: "rgba(255,255,255,0.5)", cursor: "pointer", color: "#E8899A",
      fontFamily: "inherit", fontWeight: 700, fontSize: 14, marginTop: 2, marginBottom: 16 },
    noteCard: { background: "rgba(255,255,255,0.88)", borderRadius: 16, padding: "12px 14px",
      marginBottom: 8, border: "1.5px solid #FADADD", display: "flex", gap: 10, alignItems: "flex-start" },
    noteTime: { fontSize: 11, color: "#E8A0B0", fontWeight: 700, marginBottom: 3 },
    noteTxt: { fontSize: 14, color: "#5A3040", fontWeight: 500, lineHeight: 1.5 },
    catCard: cat => ({ background: `${cat.color}55`, borderRadius: 18, padding: "14px 16px",
      marginBottom: 10, display: "flex", alignItems: "center", gap: 12, border: `2px solid ${cat.color}` }),
    statsRow: { display: "flex", gap: 10, marginBottom: 20 },
    statCard: c => ({ flex: 1, background: c, borderRadius: 18, padding: "14px", textAlign: "center" }),
    statNum: { fontSize: 26, fontWeight: 900, color: "#5A3040" },
    statLbl: { fontSize: 11, fontWeight: 700, color: "#A06070", marginTop: 2 },
    overlay: { position: "fixed", inset: 0, background: "rgba(200,100,140,0.14)", backdropFilter: "blur(4px)", zIndex: 99 },
    modal: { position: "fixed", bottom: 0, left: "50%", transform: "translateX(-50%)",
      width: "100%", maxWidth: 420, background: "#fff", borderRadius: "28px 28px 0 0",
      padding: "22px 18px 30px", boxShadow: "0 -8px 40px rgba(200,100,140,0.2)", zIndex: 100, maxHeight: "85vh", overflowY: "auto" },
    modalTit: { fontWeight: 800, fontSize: 17, color: "#C04060", marginBottom: 14 },
    inp: { width: "100%", padding: "13px 15px", borderRadius: 15, border: "2px solid #F9C6D0",
      fontFamily: "inherit", fontSize: 15, color: "#5A3040", background: "#FFF9FB",
      outline: "none", boxSizing: "border-box", marginBottom: 10 },
    sel: { width: "100%", padding: "13px 15px", borderRadius: 15, border: "2px solid #F9C6D0",
      fontFamily: "inherit", fontSize: 15, color: "#5A3040", background: "#FFF9FB",
      outline: "none", boxSizing: "border-box", marginBottom: 10, appearance: "none" },
    bigBtn: (c1, c2) => ({ width: "100%", padding: "15px", borderRadius: 17, border: "none",
      background: `linear-gradient(135deg,${c1},${c2})`, color: "#fff", fontFamily: "inherit",
      fontWeight: 800, fontSize: 15, cursor: "pointer", boxShadow: `0 6px 20px ${c1}55`, marginTop: 4 }),
    ghostBtn: { width: "100%", padding: "13px", borderRadius: 17, border: "2px solid #F9C6D0",
      background: "transparent", color: "#C08090", fontFamily: "inherit", fontWeight: 700, fontSize: 14, cursor: "pointer", marginTop: 8 },
    checkRow: { display: "flex", alignItems: "center", gap: 10, marginBottom: 12,
      padding: "11px 13px", background: "#FFF5F8", borderRadius: 13, cursor: "pointer" },
    checkbox: on => ({ width: 22, height: 22, borderRadius: 7,
      border: `2px solid ${on ? "#FFB5C8" : "#F9C6D0"}`,
      background: on ? "linear-gradient(135deg,#FFB5C8,#FFCBA4)" : "transparent",
      display: "flex", alignItems: "center", justifyContent: "center", flexShrink: 0 }),
    iconGrid: { display: "grid", gridTemplateColumns: "repeat(7,1fr)", gap: 6, marginBottom: 12 },
    iconOpt: on => ({ fontSize: 20, width: 38, height: 38, borderRadius: 11, border: "none", cursor: "pointer",
      display: "flex", alignItems: "center", justifyContent: "center",
      background: on ? "linear-gradient(135deg,#FFB5C8,#FFCBA4)" : "rgba(249,198,208,0.3)",
      boxShadow: on ? "0 3px 10px rgba(255,150,180,0.35)" : "none", transition: "all 0.2s" }),
    colorGrid: { display: "flex", gap: 8, marginBottom: 12, flexWrap: "wrap" },
    colorDot: (color, on) => ({ width: 30, height: 30, borderRadius: "50%", border: "none", cursor: "pointer",
      background: color, boxShadow: on ? `0 0 0 3px ${color}, 0 0 0 5px #fff` : "none", transition: "all 0.2s" }),
    preview: cat => ({ background: `${cat.color}60`, borderRadius: 14, padding: "12px 14px",
      marginBottom: 12, display: "flex", alignItems: "center", gap: 8, border: `2px solid ${cat.color}` }),
    navBar: { position: "fixed", bottom: 0, left: "50%", transform: "translateX(-50%)",
      width: "100%", maxWidth: 420, background: "rgba(255,255,255,0.93)", backdropFilter: "blur(16px)",
      borderTop: "1px solid #FFE0EA", padding: "10px 20px 18px", display: "flex", justifyContent: "space-around", zIndex: 50 },
    navBtn: a => ({ display: "flex", flexDirection: "column", alignItems: "center", gap: 3,
      background: a ? "linear-gradient(135deg,#FFE4EE,#FFE8D8)" : "none",
      border: "none", cursor: "pointer", padding: "6px 10px", borderRadius: 13, transition: "all 0.2s" }),
    navIcon: a => ({ fontSize: 21, filter: a ? "none" : "grayscale(0.4) opacity(0.55)" }),
    navLbl: a => ({ fontSize: 10, fontWeight: 700, color: a ? "#E8899A" : "#C0A0A8", fontFamily: "inherit" }),
  };

  const previewCat = { color: CAT_COLORS[newCatColorIdx], accent: CAT_ACCENTS[newCatColorIdx] };

  return (
    <div style={S.app}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Nunito:wght@400;500;600;700;800;900&display=swap');
        *{box-sizing:border-box;margin:0;padding:0}
        ::-webkit-scrollbar{width:0}
        .pop:active{transform:scale(0.91)!important}
        @keyframes fadeUp{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:translateY(0)}}
        .fi{animation:fadeUp 0.28s ease}
      `}</style>

      {/* HEADER */}
      <div style={S.header}>
        <div style={S.d1}/><div style={S.d2}/>
        <div style={S.dateText}>{dateStr}</div>
        <div style={S.title}>Soft & Bright 🌸</div>
        <div style={S.quoteBox}>✨ {quote}</div>
        {screen === "home" && (
          <div style={S.progRow}>
            <div style={S.progBar}>
              <div style={S.progFill(todayTasks.length ? (doneTodayCount / todayTasks.length) * 100 : 0)}/>
            </div>
            <div style={S.progTxt}>{doneTodayCount}/{todayTasks.length} місій</div>
          </div>
        )}
      </div>

      {/* HOME */}
      {screen === "home" && (<>
        <div style={S.tabs}>
          <button style={S.tab(activeTab === "today")} onClick={() => setActiveTab("today")}>📅 Сьогодні</button>
          <button style={S.tab(activeTab === "tomorrow")} onClick={() => setActiveTab("tomorrow")}>🌙 Завтра</button>
        </div>
        <div style={S.body}>
          {categories.length === 0 && (
            <div style={S.empty}>
              <div style={{ fontSize: 52, marginBottom: 10 }}>🌸</div>
              <div style={{ fontWeight: 700, fontSize: 15, color: "#C08090" }}>Привіт! Спочатку створи тему</div>
              <div style={{ fontSize: 13, marginTop: 6, color: "#D0A0B0" }}>Перейди до розділу «Теми» внизу</div>
            </div>
          )}
          {categories.length > 0 && tasksByCategory.length === 0 && (
            <div style={S.empty}>
              <div style={{ fontSize: 52, marginBottom: 10 }}>🌷</div>
              <div style={{ fontWeight: 700, fontSize: 15, color: "#C08090" }}>Завдань ще немає</div>
              <div style={{ fontSize: 13, marginTop: 6, color: "#D0A0B0" }}>Натисни кнопку нижче</div>
            </div>
          )}
          {tasksByCategory.map(({ cat, tasks: ct }) => (
            <div key={cat.id} style={{ marginBottom: 16 }} className="fi">
              <div style={S.catBadge(cat)}>
                <span style={{ fontSize: 15 }}>{cat.icon}</span>
                <span style={S.catBadgeName(cat)}>{cat.name}</span>
              </div>
              {ct.map(task => (
                <div key={task.id} style={S.taskCard(task, cat)} className="fi">
                  {task.priority && !task.done && <div style={S.prioDot}/>}
                  <div style={S.taskTxt(task.done)}>{task.text}</div>
                  {activeTab === "today" && !task.done &&
                    <button style={S.iconBtn} className="pop" onClick={() => moveTask(task.id, "tomorrow")}>→</button>}
                  {activeTab === "tomorrow" &&
                    <button style={S.iconBtn} className="pop" onClick={() => moveTask(task.id, "today")}>←</button>}
                  <button className="pop" style={S.doneBtn(task.done, cat)} onClick={() => completeTask(task.id)}>
                    {task.done ? "🌸" : "✓"}
                  </button>
                  <button className="pop" style={{ ...S.iconBtn, color: "#E8A0A0" }} onClick={() => deleteTask(task.id)}>✕</button>
                </div>
              ))}
            </div>
          ))}
          {categories.length > 0 && (
            <button style={S.addBtn} className="pop" onClick={() => { setNewTaskCat(categories[0].id); setShowAddTask(true); }}>
              + Додати місію
            </button>
          )}
        </div>
      </>)}

      {/* NOTES */}
      {screen === "notes" && (
        <div style={S.body}>
          <div style={S.secTit}>💭 Швидкі нотатки</div>
          <div style={{ display: "flex", gap: 8, marginBottom: 14 }}>
            <input style={{ ...S.inp, marginBottom: 0, flex: 1 }} placeholder="Запиши свою думку..."
              value={newNote} onChange={e => setNewNote(e.target.value)}
              onKeyDown={e => e.key === "Enter" && addNote()}/>
            <button className="pop" style={{ ...S.bigBtn("#FFB5C8", "#FFCBA4"), width: "auto", padding: "0 18px", marginTop: 0 }}
              onClick={addNote}>✦</button>
          </div>
          {notes.length === 0 && (
            <div style={S.empty}>
              <div style={{ fontSize: 52, marginBottom: 10 }}>💭</div>
              <div style={{ fontWeight: 700, fontSize: 15, color: "#C08090" }}>Тут збережуться твої думки</div>
            </div>
          )}
          {notes.map(n => (
            <div key={n.id} style={S.noteCard} className="fi">
              <div style={{ flex: 1 }}>
                <div style={S.noteTime}>🕐 {n.time}</div>
                <div style={S.noteTxt}>{n.text}</div>
              </div>
              <button className="pop" style={{ ...S.iconBtn, color: "#E8A0A0" }} onClick={() => deleteNote(n.id)}>✕</button>
            </div>
          ))}
        </div>
      )}

      {/* CATEGORIES */}
      {screen === "cats" && (
        <div style={S.body}>
          <div style={S.secTit}>🗂 Мої теми</div>
          {categories.length === 0 && (
            <div style={S.empty}>
              <div style={{ fontSize: 52, marginBottom: 10 }}>🗂</div>
              <div style={{ fontWeight: 700, fontSize: 15, color: "#C08090" }}>Ще немає жодної теми</div>
              <div style={{ fontSize: 13, marginTop: 6, color: "#D0A0B0" }}>Натисни «+ Нова тема» нижче</div>
            </div>
          )}
          {categories.map(cat => (
            <div key={cat.id} style={S.catCard(cat)} className="fi">
              <span style={{ fontSize: 26 }}>{cat.icon}</span>
              <div style={{ flex: 1 }}>
                <div style={{ fontWeight: 800, color: cat.accent, fontSize: 15 }}>{cat.name}</div>
                <div style={{ fontSize: 12, color: "#A08090", fontWeight: 600 }}>
                  {tasks.filter(t => t.catId === cat.id).length} завдань · {tasks.filter(t => t.catId === cat.id && t.done).length} виконано
                </div>
              </div>
              <button className="pop" style={{ ...S.iconBtn, color: "#E8A0A0" }} onClick={() => deleteCategory(cat.id)}>✕</button>
            </div>
          ))}
          <button style={S.addBtn} className="pop" onClick={() => setShowAddCat(true)}>+ Нова тема</button>
        </div>
      )}

      {/* STATS */}
      {screen === "stats" && (
        <div style={S.body}>
          <div style={S.secTit}>✨ Мій прогрес</div>
          <div style={S.statsRow}>
            <div style={S.statCard("#FFE4EE")}>
              <div style={S.statNum}>{tasks.filter(t => t.done).length}</div>
              <div style={S.statLbl}>ВИКОНАНО</div>
            </div>
            <div style={S.statCard("#FFE8D8")}>
              <div style={S.statNum}>{tasks.filter(t => !t.done).length}</div>
              <div style={S.statLbl}>ЗАЛИШИЛОСЬ</div>
            </div>
            <div style={S.statCard("#E8F4E8")}>
              <div style={S.statNum}>{notes.length}</div>
              <div style={S.statLbl}>НОТАТОК</div>
            </div>
          </div>
          {categories.length === 0 && (
            <div style={S.empty}>
              <div style={{ fontSize: 52, marginBottom: 10 }}>📊</div>
              <div style={{ fontWeight: 700, fontSize: 15, color: "#C08090" }}>Поки нічого немає</div>
            </div>
          )}
          {categories.map(cat => {
            const total = tasks.filter(t => t.catId === cat.id).length;
            const done = tasks.filter(t => t.catId === cat.id && t.done).length;
            const pct = total ? Math.round((done / total) * 100) : 0;
            return (
              <div key={cat.id} style={{ marginBottom: 14 }}>
                <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 5 }}>
                  <span style={{ fontWeight: 700, color: "#5A3040", fontSize: 14 }}>{cat.icon} {cat.name}</span>
                  <span style={{ fontWeight: 700, color: cat.accent, fontSize: 13 }}>{done}/{total}</span>
                </div>
                <div style={{ background: `${cat.color}45`, borderRadius: 99, height: 9, overflow: "hidden" }}>
                  <div style={{ width: `${pct}%`, height: "100%",
                    background: `linear-gradient(90deg,${cat.accent},${cat.color})`,
                    borderRadius: 99, transition: "width 0.6s ease" }}/>
                </div>
              </div>
            );
          })}
        </div>
      )}

      {/* MODAL: ADD TASK */}
      {showAddTask && (<>
        <div style={S.overlay} onClick={() => setShowAddTask(false)}/>
        <div style={S.modal}>
          <div style={S.modalTit}>✨ Нова місія</div>
          <input style={S.inp} placeholder="Що потрібно зробити?"
            value={newTaskText} onChange={e => setNewTaskText(e.target.value)}
            onKeyDown={e => e.key === "Enter" && addTask()} autoFocus/>
          <select style={S.sel} value={newTaskCat || ""} onChange={e => setNewTaskCat(Number(e.target.value))}>
            {categories.map(cat => (<option key={cat.id} value={cat.id}>{cat.icon} {cat.name}</option>))}
          </select>
          <div style={S.checkRow} onClick={() => setNewTaskPriority(p => !p)}>
            <div style={S.checkbox(newTaskPriority)}>
              {newTaskPriority && <span style={{ color: "#fff", fontSize: 13 }}>✓</span>}
            </div>
            <span style={{ fontWeight: 700, color: "#5A3040", fontSize: 14 }}>⭐ Головна місія дня</span>
          </div>
          <button className="pop" style={S.bigBtn("#FFB5C8", "#FF8FAB")} onClick={addTask}>Додати місію 🌸</button>
          <button style={S.ghostBtn} onClick={() => setShowAddTask(false)}>Скасувати</button>
        </div>
      </>)}

      {/* MODAL: ADD CATEGORY */}
      {showAddCat && (<>
        <div style={S.overlay} onClick={() => setShowAddCat(false)}/>
        <div style={S.modal}>
          <div style={S.modalTit}>🗂 Нова тема</div>
          <input style={S.inp} placeholder="Назва теми (напр. Йога, Рецепти...)"
            value={newCatName} onChange={e => setNewCatName(e.target.value)}
            onKeyDown={e => e.key === "Enter" && addCategory()} autoFocus/>
          <div style={{ fontWeight: 700, fontSize: 13, color: "#C08090", marginBottom: 8 }}>Оберіть іконку</div>
          <div style={S.iconGrid}>
            {CAT_ICONS.map(ic => (
              <button key={ic} style={S.iconOpt(newCatIcon === ic)} onClick={() => setNewCatIcon(ic)}>{ic}</button>
            ))}
          </div>
          <div style={{ fontWeight: 700, fontSize: 13, color: "#C08090", marginBottom: 8 }}>Оберіть колір</div>
          <div style={S.colorGrid}>
            {CAT_COLORS.map((color, i) => (
              <button key={color} style={S.colorDot(color, newCatColorIdx === i)} onClick={() => setNewCatColorIdx(i)}/>
            ))}
          </div>
          <div style={S.preview(previewCat)}>
            <span style={{ fontSize: 20 }}>{newCatIcon}</span>
            <span style={{ fontWeight: 800, color: previewCat.accent, fontSize: 14 }}>{newCatName || "Назва теми"}</span>
          </div>
          <button className="pop" style={S.bigBtn("#FFB5C8", "#FFCBA4")} onClick={addCategory}>Створити тему</button>
          <button style={S.ghostBtn} onClick={() => setShowAddCat(false)}>Скасувати</button>
        </div>
      </>)}

      {/* NAV */}
      <div style={S.navBar}>
        {[{ id: "home", icon: "🏠", label: "Місії" }, { id: "notes", icon: "💭", label: "Нотатки" },
          { id: "cats", icon: "🗂", label: "Теми" }, { id: "stats", icon: "✨", label: "Прогрес" }].map(nav => (
          <button key={nav.id} style={S.navBtn(screen === nav.id)} onClick={() => setScreen(nav.id)}>
            <span style={S.navIcon(screen === nav.id)}>{nav.icon}</span>
            <span style={S.navLbl(screen === nav.id)}>{nav.label}</span>
          </button>
        ))}
      </div>
    </div>
  );
}
