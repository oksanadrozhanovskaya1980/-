<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Права для подростков: Виртуальная библиотека</title>
<style>
  body { font-family: sans-serif; background: #eeeeff; margin: 0; }
  .game { max-width: 480px; margin: 2em auto; padding: 2em; background: #fff; border-radius: 12px; box-shadow: 0 0 20px #aaf2; }
  .characters { margin-bottom: 1em; }
  .map, .card-table { display: flex; gap: 6px; margin-bottom: 1em; }
  .cell { width: 42px; height: 42px; border: 1px solid #bdbdbd; display: flex; align-items: center; justify-content: center; font-size:1.2em;}
  .player { background: #3b6fc4; color: #fff; border-radius: 50%; padding: 2px 8px;}
  .npc { background: #fff0c4; border-radius: 8px; }
  button { margin: 6px 6px 0 0; padding: 0.5em 1.2em;}
  .modal { position: fixed; top:0; left:0;right:0;bottom:0; background: #0004; display:flex;align-items:center;justify-content:center; }
  .modal-content { background: #fff; padding:2em; border-radius:10px; max-width:90vw;}
  .cards {display:flex;gap:8px; flex-wrap:wrap;}
  .card {border:1px solid #999; background:#fafafc; padding:.5em 1em; border-radius:8px; margin-top:.5em;}
  .badge {background:#c7f2d1; color:#225b23;font-weight:bold; border-radius:6px; padding:2px 7px;}
</style>
</head>
<body>
<div class="game" id="root"></div>
<script>
// Простая симуляция карты: 3x3 поля с задачами
const locations = [
  [{t:"Вход", event: "intro"}, {t: "🤔", event: "law_keeper"}, {t: "Зал 1", event:null}],
  [{t: "Тест", event: "quiz"}, {t: "♻️", event:"crossword"}, {t:"Отд. Закона", event:null}],
  [{t:"Здоровье", event: "health"}, {t:"🔥", event: "finale"}, {t:"Читальный зал",event:null}],
];

const cardsInfo = [
  {tag:"право на образование", text:"Каждый имеет право на образование."},
  {tag:"свобода мнений", text:"Гарантируется свобода выражения мнений."},
  {tag:"право на медпомощь", text:"Дети имеют право на бесплатную мед.помощь."},
];

function App(){
  const [player, setPlayer] = React.useState({x:0,y:0});
  const [modal, setModal] = React.useState(null);
  const [cards, setCards] = React.useState([]);
  const [xp, setXp] = React.useState(0);

  // simple level: xp<50 - новичок, <100 - исследователь, >=100 - профи
  let level = "Новичок";
  if(xp>90) level="Профессионал"; else if(xp>40) level="Исследователь";

  function move(dx, dy){
    let nx = Math.max(0, Math.min(2, player.x+dx));
    let ny = Math.max(0, Math.min(2, player.y+dy));
    setPlayer({x:nx, y:ny});
    let loc = locations[ny][nx];
    if(loc.event){
      handleEvent(loc.event);
    }
  }

  function handleEvent(event){
    if(event === "intro") setModal({
      title:"Привет, Артем!",
      content:
        <>Ты в необычной библиотеке.<br/>Твоя задача: собрать все карточки с правами и ответить на вопросы духов-загадочников!</>,
      btn:"Погнали!", onClose:()=>setModal(null)
    });
    else if(event==="law_keeper") setModal({
      title:"Первый загадочник",
      content:<>Вход закрыт. Чтобы пройти дальше, подбери ключ к замку.<br/><b>Вопрос:</b> Выбери правильный ключ<br/>
        <div style={{marginTop:8}}>
          <button onClick={()=>handleResult(0,false)}>Ключ с рисунком книги</button>
          <button onClick={()=>handleResult(0,true)}>Ключ Конституции</button>
        </div>
      </>,
      btn:null
    });
    else if(event==="quiz") setModal({
      title:"Дух Закона",
      content:<>Какая книга — основной закон РФ?<br/>
        <div style={{marginTop:8}}>
          <button onClick={()=>handleResult(1,false)}>Уголовный кодекс</button>
          <button onClick={()=>handleResult(1,true)}>Конституция</button>
        </div>
      </>,
      btn:null
    });
    else if(event==="crossword") setModal({
      title:"Мини-кроссворд",
      content:<>Вопрос по горизонтали: Документ, закрепляющий права...<br/>
        <input id="cw1" placeholder="Введите слово"/><br/>
        Вопрос по вертикали: Свобода мыслей — это...<br/>
        <input id="cw2" placeholder="Введите слово"/><br/>
        <button onClick={()=>handleResult(2,(document.getElementById('cw1').value.trim().toLowerCase()==="конституция"&&document.getElementById('cw2').value.trim().toLowerCase()==="мысль"))}>Ответить</button>
      </>,
      btn:null
    });
    else if(event==="health") setModal({
      title:"Здоровье и безопасность",
      content:<>Выберите верные утверждения:<br/>
        <label><input type="checkbox" id="ch1"/>Школьники обязаны проходить медосмотр</label><br/>
        <label><input type="checkbox" id="ch2"/>Родители вправе отказаться от прививок</label><br/>
        <label><input type="checkbox" id="ch3"/>Медпомощь только платная</label><br/>
        <label><input type="checkbox" id="ch4"/>Дети имеют право на консультацию психолога</label><br/>
        <label><input type="checkbox" id="ch5"/>Лечение только за деньги</label><br/>
        <label><input type="checkbox" id="ch6"/>Школы проводят бесплатные осмотры</label><br/>
        <button onClick={()=>handleResult(3,checkHealth())}>Проверить</button>
      </>,
      btn:null
    });
    else if(event==="finale") setModal({
      title:"Финал",
      content:
        <>Поздравляем! Ты собрал все карточки и стал экспертом по правам!<br/>
        <span class="badge">🏆 Профессионал!</span></>,
      btn:"Завершить", onClose:()=>setModal(null)
    });
  }

  function handleResult(idx, ok){
    setModal({
      title: ok?"Правильно!":"Ошибка",
      content: ok?cardsInfo[idx] ?<><div className="card">{cardsInfo[idx].text}</div>Ты получаешь карточку: <b>{cardsInfo[idx].tag}</b></> : <>Ты молодец!</>:<>Попробуй еще раз!</>,
      btn:"Продолжить",
      onClose:()=>{
        setModal(null);
        if(ok && cardsInfo[idx] && !cards.find(c=>c.tag==cardsInfo[idx].tag)){
          setCards([...cards,cardsInfo[idx]]);
          setXp(xp+40);
        }
      }
    });
  }

  function checkHealth(){
    return (
      document.getElementById('ch1').checked &&
      document.getElementById('ch2').checked &&
      !document.getElementById('ch3').checked &&
      document.getElementById('ch4').checked &&
      !document.getElementById('ch5').checked &&
      document.getElementById('ch6').checked
    );
  }

  return (
    <div>
      <div className="characters">Ты — <b>Артем</b> <span className="badge">{level}</span></div>
      <div className="map">
        {locations.map((row, y)=>
          <div key={y} style={{display:'flex', flexDirection:'column'}}>
            {row.map((loc,x)=>
              <div className="cell" key={x} style={{borderColor:(player.x==x&&player.y==y)?'#3b6fc4':'#bdbdbd'}}>
                {player.x==x && player.y==y? <span className="player">🙂</span> : null}
                {loc.event&&! (player.x==x&&player.y==y) ? <span className="npc">{loc.t}</span>: null}
              </div>
            )}
          </div>
        )}
      </div>
      <div>
        <b>Управление:</b><br/>
        <button onClick={()=>move(0,-1)}>⬆️</button>
        <button onClick={()=>move(-1,0)}>⬅️</button>
        <button onClick={()=>move(1,0)}>➡️</button>
        <button onClick={()=>move(0,1)}>⬇️</button>
      </div>
      <div className="cards">
        {cards.map(card=><div className="card" key={card.tag}>{card.text}</div>)}
      </div>
      <div style={{marginTop:12}}>Очки опыта: <b>{xp}</b></div>
      {modal?
        <div className="modal"><div className="modal-content"><h3>{modal.title}</h3>
        {modal.content}
        {modal.btn?<div><button onClick={modal.onClose || (()=>setModal(null))}>{modal.btn}</button></div>:null}</div></div>
      :null}
    </div>
  );
}

// Подключение React через CDN >>>
const script = document.createElement('script');
script.src = 'https://unpkg.com/react@18/umd/react.development.js';
document.body.appendChild(script);
const scriptd = document.createElement('script');
scriptd.src = 'https://unpkg.com/react-dom@18/umd/react-dom.development.js';
document.body.appendChild(scriptd);
scriptd.onload = ()=>{
  ReactDOM.createRoot(document.getElementById('root')).render(React.createElement(App));
}
</script>
</body>
</html>
