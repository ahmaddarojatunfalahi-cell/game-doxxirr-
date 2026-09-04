```react
import React, { useState, useEffect } from 'react';

// Aturan Ular dan Tangga (Kunci: Posisi Awal, Nilai: Posisi Akhir)
const SNAKES_AND_LADDERS = {
  // Tangga (Naik) - Key = Awal, Value = Akhir
  2: 38, 7: 14, 8: 31, 15: 26, 21: 42, 28: 84, 36: 44, 51: 67, 71: 91, 78: 98, 87: 94,
  // Ular (Turun) - Key = Kepala (Awal), Value = Ekor (Akhir)
  16: 6, 46: 25, 49: 11, 62: 19, 64: 60, 74: 53, 89: 68, 92: 88, 95: 75, 99: 80
};

// Fungsi untuk membuat papan ular tangga (pola zig-zag)
const generateBoard = () => {
  let board = [];
  for (let row = 9; row >= 0; row--) {
    let rowCells = [];
    for (let col = 1; col <= 10; col++) {
      rowCells.push(row * 10 + col);
    }
    if (row % 2 !== 0) rowCells.reverse();
    board.push(...rowCells);
  }
  return board;
};

// Fungsi untuk mendapatkan kordinat (X, Y) tengah dari sebuah kotak (Skala 0-100)
const getCellCenter = (cell) => {
  const row = Math.floor((cell - 1) / 10);
  const isRightToLeft = row % 2 !== 0;
  const col = (cell - 1) % 10;
  const actualCol = isRightToLeft ? 9 - col : col;
  return {
    x: (actualCol * 10) + 5,
    y: ((9 - row) * 10) + 5
  };
};

// Komponen Wajah Dadu
const Dice = ({ value, isRolling }) => {
  const getDots = (val) => {
    switch (val) {
      case 1: return <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-2 row-start-2"></div>;
      case 2: return (
        <>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-3 row-start-1"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-1 row-start-3"></div>
        </>
      );
      case 3: return (
        <>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-3 row-start-1"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-2 row-start-2"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-1 row-start-3"></div>
        </>
      );
      case 4: return (
        <>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-1 row-start-1"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-3 row-start-1"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-1 row-start-3"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-3 row-start-3"></div>
        </>
      );
      case 5: return (
        <>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-1 row-start-1"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-3 row-start-1"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-2 row-start-2"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-1 row-start-3"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-3 row-start-3"></div>
        </>
      );
      case 6: return (
        <>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-1 row-start-1"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-3 row-start-1"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-1 row-start-2"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-3 row-start-2"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-1 row-start-3"></div>
          <div className="w-4 h-4 sm:w-5 sm:h-5 bg-slate-800 rounded-full col-start-3 row-start-3"></div>
        </>
      );
      default: return null;
    }
  };

  return (
    <div className={`w-20 h-20 sm:w-28 sm:h-28 bg-white border-4 border-slate-800 rounded-2xl shadow-[4px_4px_0px_rgba(30,41,59,1)] flex items-center justify-center p-3 transition-transform duration-75 ${isRolling ? 'rotate-12 scale-110' : 'rotate-0 scale-100'}`}>
      {value ? (
        <div className="grid grid-cols-3 grid-rows-3 gap-1 w-full h-full place-items-center">
          {getDots(value)}
        </div>
      ) : (
        <span className="text-4xl sm:text-6xl">🎲</span>
      )}
    </div>
  );
};

export default function App() {
  const [board] = useState(generateBoard());
  const [player1Pos, setPlayer1Pos] = useState(1);
  const [player2Pos, setPlayer2Pos] = useState(1);
  const [turn, setTurn] = useState(1);
  const [dice, setDice] = useState(null);
  const [message, setMessage] = useState("Mulai permainan! Giliran Pemain 1 (Merah).");
  const [gameOver, setGameOver] = useState(false);
  const [isAnimating, setIsAnimating] = useState(false);

  const rollDice = () => {
    if (gameOver || isAnimating) return;
    
    setIsAnimating(true);
    let rollCount = 0;
    const finalRoll = Math.floor(Math.random() * 6) + 1;

    const rollInterval = setInterval(() => {
      setDice(Math.floor(Math.random() * 6) + 1);
      rollCount++;
      
      if (rollCount > 10) {
        clearInterval(rollInterval);
        setDice(finalRoll); 
        
        setTimeout(() => {
          movePlayer(finalRoll);
          setIsAnimating(false);
        }, 500); 
      }
    }, 80);
  };

  const movePlayer = (roll) => {
    const currentPlayerPos = turn === 1 ? player1Pos : player2Pos;
    let newPos = currentPlayerPos + roll;

    if (newPos > 100) {
      setMessage(`Pemain ${turn} mendapat ${roll}, butuh angka pas untuk menang!`);
      switchTurn();
      return;
    }

    let finalPos = newPos;
    let actionMessage = "";

    if (SNAKES_AND_LADDERS[newPos]) {
      finalPos = SNAKES_AND_LADDERS[newPos];
      if (finalPos > newPos) {
        actionMessage = ` 🪜 Naik Tangga ke ${finalPos}!`;
      } else {
        actionMessage = ` 🐍 Digigit Ular, turun ke ${finalPos}!`;
      }
    }

    if (turn === 1) setPlayer1Pos(finalPos);
    else setPlayer2Pos(finalPos);

    if (finalPos === 100) {
      setMessage(`Selamat! Pemain ${turn} Menang! 🎉`);
      setGameOver(true);
    } else {
      setMessage(`Pemain ${turn} maju ke ${newPos}.${actionMessage}`);
      switchTurn();
    }
  };

  const switchTurn = () => setTurn((prev) => (prev === 1 ? 2 : 1));

  const resetGame = () => {
    setPlayer1Pos(1);
    setPlayer2Pos(1);
    setTurn(1);
    setDice(null);
    setGameOver(false);
    setMessage("Permainan diulang! Giliran Pemain 1 (Merah).");
  };

  // SVG Renderer untuk Tangga
  const renderLadder = (start, end) => {
    const p1 = getCellCenter(start);
    const p2 = getCellCenter(end);
    const dx = p2.x - p1.x;
    const dy = p2.y - p1.y;
    const len = Math.sqrt(dx * dx + dy * dy);
    
    // Normal vector untuk jarak antar tiang tangga
    const nx = (dy / len) * 1.8;
    const ny = (-dx / len) * 1.8;

    const steps = Math.floor(len / 4); 
    const rungs = [];
    for(let i = 1; i < steps; i++) {
       const t = i / steps;
       const cx = p1.x + dx * t;
       const cy = p1.y + dy * t;
       rungs.push(
           <line key={i} x1={cx + nx} y1={cy + ny} x2={cx - nx} y2={cy - ny} stroke="#b45309" strokeWidth="0.8" />
       );
    }

    return (
      <g key={`ladder-${start}`}>
        <line x1={p1.x + nx} y1={p1.y + ny} x2={p2.x + nx} y2={p2.y + ny} stroke="#92400e" strokeWidth="1.5" strokeLinecap="round" />
        <line x1={p1.x - nx} y1={p1.y - ny} x2={p2.x - nx} y2={p2.y - ny} stroke="#92400e" strokeWidth="1.5" strokeLinecap="round" />
        {rungs}
      </g>
    );
  };

  // SVG Renderer untuk Ular
  const renderSnake = (start, end) => {
    const p1 = getCellCenter(start); // Kepala
    const p2 = getCellCenter(end);   // Ekor
    
    const dx = p2.x - p1.x;
    const dy = p2.y - p1.y;
    
    // Membuat lekukan tubuh (Kurva Bezier)
    const cx1 = p1.x + dx * 0.2 - dy * 0.4;
    const cy1 = p1.y + dy * 0.2 + dx * 0.4;
    const cx2 = p1.x + dx * 0.8 + dy * 0.4;
    const cy2 = p1.y + dy * 0.8 - dx * 0.4;

    return (
      <g key={`snake-${start}`}>
        {/* Shadow/Garis Luar */}
        <path d={`M ${p1.x} ${p1.y} C ${cx1} ${cy1}, ${cx2} ${cy2}, ${p2.x} ${p2.y}`} fill="none" stroke="#064e3b" strokeWidth="3" strokeLinecap="round" />
        {/* Badan Ular */}
        <path d={`M ${p1.x} ${p1.y} C ${cx1} ${cy1}, ${cx2} ${cy2}, ${p2.x} ${p2.y}`} fill="none" stroke="#22c55e" strokeWidth="2" strokeLinecap="round" />
        {/* Motif Sisik */}
        <path d={`M ${p1.x} ${p1.y} C ${cx1} ${cy1}, ${cx2} ${cy2}, ${p2.x} ${p2.y}`} fill="none" stroke="#166534" strokeWidth="2" strokeDasharray="1.5 2.5" strokeLinecap="round" />

        {/* Kepala Ular */}
        <circle cx={p1.x} cy={p1.y} r="2.2" fill="#22c55e" />
        <circle cx={p1.x} cy={p1.y} r="2.2" fill="none" stroke="#064e3b" strokeWidth="0.4" />
        
        {/* Mata */}
        <circle cx={p1.x - 0.8} cy={p1.y - 0.8} r="0.6" fill="white" />
        <circle cx={p1.x + 0.8} cy={p1.y - 0.8} r="0.6" fill="white" />
        <circle cx={p1.x - 0.8} cy={p1.y - 0.8} r="0.3" fill="black" />
        <circle cx={p1.x + 0.8} cy={p1.y - 0.8} r="0.3" fill="black" />
        
        {/* Lidah */}
        <path d={`M ${p1.x} ${p1.y+2} L ${p1.x} ${p1.y+3.5} L ${p1.x-0.8} ${p1.y+4.5} M ${p1.x} ${p1.y+3.5} L ${p1.x+0.8} ${p1.y+4.5}`} stroke="#ef4444" strokeWidth="0.3" fill="none" />
      </g>
    );
  };

  return (
    <div className="min-h-screen bg-slate-900 flex flex-col xl:flex-row font-sans overflow-hidden">
      
      {/* Area Papan (Memaksimalkan ukuran, menjaga rasio kotak) */}
      <div className="flex-grow flex items-center justify-center p-2 sm:p-4 bg-slate-800/80 relative">
        
        {/* Container Papan: aspect-square menjamin bentuk selalu kotak sempurna */}
        <div className="relative w-full max-w-[100vmin] xl:max-w-none xl:h-[95vh] xl:w-[95vh] aspect-square bg-white rounded-xl shadow-[0_0_40px_rgba(0,0,0,0.5)] border-[6px] sm:border-[10px] border-slate-700 mx-auto">
          
          {/* Label Judul */}
          <div className="absolute -top-5 sm:-top-7 left-1/2 -translate-x-1/2 bg-yellow-400 text-slate-900 px-4 py-1 sm:px-8 sm:py-2 rounded-full font-black border-4 border-slate-900 shadow-[4px_4px_0_rgba(15,23,42,1)] z-30 whitespace-nowrap text-sm sm:text-xl md:text-2xl tracking-tight">
             ULAR TANGGA DOXXIR
          </div>

          {/* LAYER 1: Grid Kotak Dasar dan Angka */}
          <div className="absolute inset-0 grid grid-cols-10 grid-rows-10 w-full h-full z-0 rounded-lg overflow-hidden">
            {board.map((cellNum) => {
              const isLadder = Object.keys(SNAKES_AND_LADDERS).includes(String(cellNum)) && SNAKES_AND_LADDERS[cellNum] > cellNum;
              const isSnake = Object.keys(SNAKES_AND_LADDERS).includes(String(cellNum)) && SNAKES_AND_LADDERS[cellNum] < cellNum;
              const isEvenCell = cellNum % 2 === 0;

              return (
                <div
                  key={`cell-${cellNum}`}
                  className={`relative border-r border-b sm:border-r-2 sm:border-b-2 border-slate-200 flex items-center justify-center p-1
                    ${isSnake ? 'bg-red-50/70' : isLadder ? 'bg-emerald-50/70' : isEvenCell ? 'bg-amber-50/40' : 'bg-white'}
                  `}
                >
                  <span className="absolute top-0 left-0.5 sm:top-1 sm:left-1 text-[8px] sm:text-xs md:text-sm text-slate-400 font-bold select-none">
                    {cellNum}
                  </span>
                </div>
              );
            })}
          </div>

          {/* LAYER 2: Gambar Ular dan Tangga (SVG) */}
          {/* viewBox="0 0 100 100" membuat sistem kordinat menjadi persentase otomatis */}
          <svg className="absolute inset-0 w-full h-full pointer-events-none z-10 drop-shadow-[0_4px_6px_rgba(0,0,0,0.6)]" viewBox="0 0 100 100" preserveAspectRatio="none">
             {Object.entries(SNAKES_AND_LADDERS).filter(([k, v]) => parseInt(v) > parseInt(k)).map(([k, v]) => renderLadder(parseInt(k), parseInt(v)))}
             {Object.entries(SNAKES_AND_LADDERS).filter(([k, v]) => parseInt(v) < parseInt(k)).map(([k, v]) => renderSnake(parseInt(k), parseInt(v)))}
          </svg>

          {/* LAYER 3: Bidak Pemain */}
          <div className="absolute inset-0 grid grid-cols-10 grid-rows-10 w-full h-full z-20 pointer-events-none">
            {board.map((cellNum) => {
              const p1Here = player1Pos === cellNum;
              const p2Here = player2Pos === cellNum;

              return (
                <div key={`token-${cellNum}`} className="relative flex items-center justify-center p-1">
                  <div className="flex flex-wrap gap-1 z-10 justify-center items-center w-full pointer-events-auto">
                    {p1Here && (
                      <div className="w-4 h-4 sm:w-6 sm:h-6 lg:w-8 lg:h-8 bg-red-600 rounded-full shadow-[0_4px_10px_rgba(220,38,38,0.8)] border-[2px] sm:border-[3px] border-white animate-bounce" title="Pemain 1" />
                    )}
                    {p2Here && (
                      <div className="w-4 h-4 sm:w-6 sm:h-6 lg:w-8 lg:h-8 bg-blue-600 rounded-full shadow-[0_4px_10px_rgba(37,99,235,0.8)] border-[2px] sm:border-[3px] border-white animate-bounce" style={{animationDelay: '0.1s'}} title="Pemain 2" />
                    )}
                  </div>
                </div>
              );
            })}
          </div>

        </div>
      </div>

      {/* Area Kontrol & Info (Menyesuaikan dengan mode mobile / dekstop) */}
      <div className="w-full xl:w-[400px] xl:min-h-screen bg-slate-900 text-slate-100 p-4 sm:p-6 flex flex-col gap-4 sm:gap-6 shadow-[-10px_0_20px_rgba(0,0,0,0.4)] z-10 overflow-y-auto">
        
        {/* Status Box */}
        <div className="bg-slate-800 border border-slate-700 p-3 sm:p-4 rounded-xl text-center shadow-lg">
          <h2 className="text-lg sm:text-xl font-black mb-1 sm:mb-2 text-yellow-400 tracking-wide uppercase">Status</h2>
          <p className="text-xs sm:text-base font-medium min-h-[3rem] sm:min-h-[3.5rem] flex items-center justify-center text-slate-300">
            {message}
          </p>
        </div>

        {/* Panel Pemain */}
        <div className="flex justify-between items-center bg-slate-800 border border-slate-700 p-3 sm:p-4 rounded-xl shadow-lg">
          <div className={`flex flex-col items-center p-2 sm:p-3 rounded-lg transition-all w-[48%] ${turn === 1 ? 'bg-red-500/20 border border-red-500 scale-105 shadow-[0_0_15px_rgba(239,68,68,0.3)]' : 'opacity-40'}`}>
            <div className="w-6 h-6 sm:w-8 sm:h-8 bg-red-600 rounded-full mb-1 sm:mb-2 shadow-md border-2 border-white"></div>
            <span className="text-[10px] sm:text-xs font-bold text-slate-300">Pemain 1</span>
            <span className="text-lg sm:text-2xl font-black text-red-400">Pos: {player1Pos}</span>
          </div>
          <div className={`flex flex-col items-center p-2 sm:p-3 rounded-lg transition-all w-[48%] ${turn === 2 ? 'bg-blue-500/20 border border-blue-500 scale-105 shadow-[0_0_15px_rgba(59,130,246,0.3)]' : 'opacity-40'}`}>
            <div className="w-6 h-6 sm:w-8 sm:h-8 bg-blue-600 rounded-full mb-1 sm:mb-2 shadow-md border-2 border-white"></div>
            <span className="text-[10px] sm:text-xs font-bold text-slate-300">Pemain 2</span>
            <span className="text-lg sm:text-2xl font-black text-blue-400">Pos: {player2Pos}</span>
          </div>
        </div>

        {/* Area Aksi & Dadu */}
        <div className="flex flex-row xl:flex-col justify-around xl:justify-center items-center gap-4 xl:gap-6 mt-2 xl:mt-auto xl:mb-10 w-full">
          
          <div onClick={rollDice} className={`cursor-pointer ${isAnimating ? 'pointer-events-none' : ''}`}>
            <Dice value={dice} isRolling={isAnimating} />
          </div>

          <div className="w-full xl:w-full max-w-[200px] xl:max-w-none">
            {!gameOver ? (
              <button
                onClick={rollDice}
                disabled={isAnimating}
                className={`w-full py-3 sm:py-4 rounded-xl text-white font-black text-sm sm:text-lg transition-all uppercase tracking-wider
                  ${turn === 1 ? 'bg-red-600 hover:bg-red-500 shadow-[0_6px_0px_rgba(153,27,27,1)]' : 'bg-blue-600 hover:bg-blue-500 shadow-[0_6px_0px_rgba(30,58,138,1)]'}
                  ${isAnimating ? 'opacity-70 cursor-not-allowed scale-95 shadow-none translate-y-1.5' : 'active:scale-95 active:shadow-none active:translate-y-1.5'}
                `}
              >
                {isAnimating ? 'Mengocok...' : `Kocok Dadu`}
              </button>
            ) : (
              <button
                onClick={resetGame}
                className="w-full py-3 sm:py-4 rounded-xl bg-emerald-500 hover:bg-emerald-400 text-white font-black text-sm sm:text-lg uppercase tracking-wider transition-all shadow-[0_6px_0px_rgba(6,78,59,1)] active:scale-95 active:shadow-none active:translate-y-1.5"
              >
                Main Lagi 🔄
              </button>
            )}
          </div>

        </div>
        
      </div>
    </div>
  );
}

```
