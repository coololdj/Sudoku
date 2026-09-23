あなたは、提出された数独画像から、スマートフォン向けの単一HTML数独アプリを生成・検証する。

目的：数独として正確で、日本人向けに見やすく、スマートフォンで操作しやすく、共有まで安定して動作する、マイクロコードのHTMLを作る。

【最重要原則】
- 正しい完成盤面を誤答扱いしない。
- 修正は継ぎ足しではなく置換し、旧ロジック・旧データを残さない。
- 同一目的の正解データを複数持たない。
- 固定問題では、完成HTMLに不要なソルバーを残さない。
- 出力前に正常系・異常系の回帰テストを行う。
- 見本コード "sudoku_mihon.html" をベースにする。
-【画像読み取りデータ部】const START= は画像読み取りデーターに差し替える。

【盤面読取り・検証】
- 画像から9×9盤面を読み取り、空白は0とする。
- 不確実なセルは推測しない。
- 初期盤面について、行・列・3×3ブロックの矛盾を確認する。
- 生成時に解の存在と原則一意性を確認し、固定数字との整合を検証する。
- 一意解でない場合は通常の完成HTMLを生成しない。

【正答判定】
- 一意解確認済みの固定問題では、実行時の正答判定は原則として「空欄なし＋行・列・3×3の重複なし」とする。
- 事前生成した正解配列との完全一致だけで正答を拒否しない。
- 正解配列を保持する場合も1個だけとする。

【機能】
- 1～9入力、消去、固定数字変更禁止
- 行・列・3×3の重複をリアルタイム検出
- 答え合わせ
- `performance.now()` による1秒単位のタイマー
- PCでは1～9、0、Backspace、Deleteにも対応
- 正解後は入力停止

表示：
未入力「未入力のマスがあります。」
誤り「誤りがあります。」
正解「正解です！ クリアタイム: MM:SS」

【UI】
- スマートフォン最優先、日本語UI、幅360～390px程度。
- 数独盤面は「81個のカード」ではなく「1枚の9×9方眼」とする。
- セル間gapや盤面セルの丸角は使わない。
- 通常線は細い明灰色、3×3境界と外周は濃い太線で連続表示する。
- 盤面構造線とセル状態表示を分離する。
- 固定数字は濃色、入力数字は藍系、選択は淡青、エラーは淡赤。
- 操作ボタンは適度な丸角可。
- 過度な影・原色・グラデーションは避ける。
- `user-scalable=no` やページ全体の `user-select:none` は使わない。

【状態管理】
- 固定構造は初期化時に一度だけ作る。
- renderでは選択・ハイライト・エラー等の状態だけ更新する。
- 同じ処理や境界判定を複数箇所へ重複実装しない。

【共有】
スマートフォン：
- 対応時のみWeb Share APIでテキスト＋PNGを共有する。
- 画像は正解時に事前生成し、準備完了後に共有可能にする。
- キャンセルはそのまま終了し、別共有へ自動fallbackしない。

Windows PC：
- Web Shareファイル共有やLINE URL scheme、ローカル `.lnk` / `.exe` の直接起動に依存しない。
- 「結果文をコピー」と「結果画像を保存」を提供する。
- LINE自動起動はHTML単体の必須要件としない。

【コード品質】
- HTML/CSS/JavaScriptを1ファイルにまとめる。
- Vanilla JavaScriptのみ。外部ライブラリ不要。
- 不要な配列・関数・コメント・デバッグコードを残さない。
- 短さより正確性を優先するが、正確性を保てる範囲で最小化する。
- 修正時は旧処理を削除して置換する。

【出力前テスト】
最低限、以下を確認する。
- 正しい完成盤面 → 正解
- 1セル空欄 → 未入力
- 行・列・3×3の各重複 → 誤り
- 初期盤面と解が矛盾しない
- 共有・タイマー・入力が主要経路で動作する
- HTML/CSS/JavaScriptがそのまま保存・実行できる

設計原則：
正しさを先に固定し、UIを載せ、最後にコードを削る。
数独盤面は方眼、操作部はアプリUI。
正答判定はルールベース。
修正は追加ではなく置換。

以下に見本のコード "sudoku_mihon.html" を示す。
<!doctype html>
<html lang="ja">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>数独</title>
<style>
*{box-sizing:border-box}
body{margin:0;padding:12px;background:#f8f9fa;color:#333;font-family:-apple-system,BlinkMacSystemFont,"Segoe UI","Hiragino Sans","Yu Gothic UI",Roboto,sans-serif;-webkit-tap-highlight-color:transparent}
main{max-width:360px;margin:auto}
header{display:flex;justify-content:space-between;align-items:center;margin:0 4px 12px}
h1{margin:0;font-size:1.25rem}
#timer{padding:4px 10px;border-radius:6px;background:#e9ecef;font:700 1.1rem monospace}
#wrap{position:relative;width:100%;aspect-ratio:1;margin-bottom:12px;background:#fff;border:3px solid #343a40;overflow:hidden}
#board{display:grid;grid-template-columns:repeat(9,1fr);width:100%;height:100%}
.cell{display:flex;align-items:center;justify-content:center;border:0;border-right:1px solid #dee2e6;border-bottom:1px solid #dee2e6;padding:0;background:#fff;color:#315f8a;font-size:1.35rem;font-weight:700;outline:0;cursor:pointer}
.cell:nth-child(9n){border-right:0}.cell:nth-child(n+73){border-bottom:0}
.fixed{background:#f2f4f6;color:#212529;font-weight:800;cursor:default}
.grid{position:absolute;inset:0;pointer-events:none;display:grid;grid-template:repeat(3,1fr)/repeat(3,1fr)}
.grid i:nth-child(3n+1),.grid i:nth-child(3n+2){border-right:3px solid #343a40}
.grid i:nth-child(-n+6){border-bottom:3px solid #343a40}
.sel{background:#b3d7ff!important}.hi{background:#f0f4f8}.same{background:#d0e2ff}.err{background:#f8d7da!important;color:#b02a37!important}
#status{min-height:24px;margin-bottom:12px;text-align:center;color:#b02a37;font-weight:700;font-size:.95rem}.ok{color:#2f7a45!important}
#pad{display:grid;grid-template-columns:repeat(5,1fr);gap:8px;margin-bottom:12px}
.btn{border:1px solid #ced4da;border-radius:8px;background:#fff;font-weight:700;cursor:pointer;touch-action:manipulation}
.num{padding:12px 0;font-size:1.2rem}.clear{background:#f8d7da;color:#721c24;border-color:#f5c6cb;font-size:.95rem}
.action{width:100%;padding:14px;border:0;border-radius:8px;background:#526f8b;color:#fff;font-size:1.05rem;font-weight:700;cursor:pointer}
.sub{margin-top:6px;padding:10px;background:#6c757d;font-size:.95rem}
#shareMobile{background:#06c755}#line{background:#00b900}
.btn:focus-visible{outline:3px solid #80bdff}
#share{display:none;margin-top:8px}[hidden],canvas{display:none!important}
</style>
</head>
<body>
<main>
<header><h1>数独</h1><div id="timer">00:00</div></header>

<div id="wrap">
<div id="board"></div>
<div class="grid"><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i></div>
</div>

<div id="status" aria-live="polite"></div>

<div id="pad">
<button class="btn num" data-n="1">1</button><button class="btn num" data-n="2">2</button><button class="btn num" data-n="3">3</button><button class="btn num" data-n="4">4</button><button class="btn num" data-n="5">5</button>
<button class="btn num" data-n="6">6</button><button class="btn num" data-n="7">7</button><button class="btn num" data-n="8">8</button><button class="btn num" data-n="9">9</button><button class="btn num clear" data-n="0">消去</button>
</div>

<button id="check" class="action btn">答え合わせ</button>

<div id="share">
<button id="shareMobile" class="action btn" hidden>結果を共有</button>
<button id="line" class="action btn sub">LINEで送る</button>
<button id="copy" class="action btn sub">結果文をコピー</button>
<button id="save" class="action btn sub">結果画像を保存</button>
</div>
</main>

<canvas id="cv" width="600" height="700"></canvas>

<script>
const START=[
[0,0,0,0,0,0,0,0,0],[0,0,0,0,0,0,0,0,0],[0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0],[0,0,0,0,0,0,0,0,0],[0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0],[0,0,0,0,0,0,0,0,0],[0,0,0,0,0,0,0,0,0]
];

const $=s=>document.getElementById(s),
board=$("board"),status=$("status"),timer=$("timer"),
check=$("check"),share=$("share"),
shareMobile=$("shareMobile"),cv=$("cv");

let a=START.map(r=>[...r]),
sel=null,done=false,file=null,start=performance.now();

const cell=(r,c)=>board.children[r*9+c],
time=()=>{
 let t=Math.floor((performance.now()-start)/1000);
 return`${String(t/60|0).padStart(2,"0")}:${String(t%60).padStart(2,"0")}`
};

function init(){
 START.forEach((row,r)=>row.forEach((n,c)=>{
  let e=document.createElement(n?"div":"button");
  e.className="cell"+(n?" fixed":"");
  e.textContent=n||"";

  if(!n){
   e.type="button";
   e.ariaLabel=`${r+1}行${c+1}列`;
   e.onclick=()=>{
    sel={r,c};
    render()
   }
  }
  board.append(e)
 }))
}

function conflicts(){
 let s=new Set;

 for(let r=0;r<9;r++)
  for(let c=0;c<9;c++){
   let n=a[r][c];
   if(!n)continue;

   for(let i=0;i<9;i++)
    if((i!=c&&a[r][i]==n)||(i!=r&&a[i][c]==n)){
     s.add(r+"-"+c);
     break
    }

   let R=r-r%3,C=c-c%3;

   for(let y=R;y<R+3;y++)
    for(let x=C;x<C+3;x++)
     if((y!=r||x!=c)&&a[y][x]==n)
      s.add(r+"-"+c)
  }

 return s
}

function render(){
 let er=conflicts(),
 v=sel?a[sel.r][sel.c]:0;

 for(let r=0;r<9;r++)
  for(let c=0;c<9;c++){
   let e=cell(r,c),n=a[r][c];

   e.classList.remove("sel","hi","same","err");

   if(sel){
    let b=
     (r/3|0)==(sel.r/3|0)&&
     (c/3|0)==(sel.c/3|0);

    if(r==sel.r&&c==sel.c)
     e.classList.add("sel");
    else if(r==sel.r||c==sel.c||b)
     e.classList.add("hi");

    if(v&&n==v)e.classList.add("same")
   }

   if(er.has(r+"-"+c))
    e.classList.add("err")
  }
}

function input(n){
 if(!sel||done)return;

 let{r,c}=sel;
 a[r][c]=n;
 cell(r,c).textContent=n||"";

 status.textContent="";
 status.classList.remove("ok");
 render()
}

function tick(){
 if(!done)timer.textContent=time()
}

function draw(){
 let x=cv.getContext("2d"),
 p=30,t=120,z=540,w=60;

 x.fillStyle="#f8f9fa";
 x.fillRect(0,0,600,700);

 x.textAlign="center";
 x.textBaseline="middle";

 x.fillStyle="#212529";
 x.font="bold 32px sans-serif";
 x.fillText("数独クリア！",300,50);

 x.fillStyle="#526f8b";
 x.font="bold 24px monospace";
 x.fillText("タイム: "+timer.textContent,300,90);

 for(let r=0;r<9;r++)
  for(let c=0;c<9;c++){
   let X=p+c*w,Y=t+r*w,f=START[r][c]!==0;

   x.fillStyle=f?"#f2f4f6":"#fff";
   x.fillRect(X,Y,w,w);

   x.strokeStyle="#dee2e6";
   x.lineWidth=1;
   x.strokeRect(X,Y,w,w);

   x.fillStyle=f?"#212529":"#315f8a";
   x.font="bold 32px sans-serif";
   x.fillText(a[r][c],X+w/2,Y+w/2)
  }

 x.strokeStyle="#343a40";
 x.lineWidth=3;

 for(let i=0;i<=9;i+=3){
  x.beginPath();
  x.moveTo(p+i*w,t);
  x.lineTo(p+i*w,t+z);
  x.stroke();

  x.beginPath();
  x.moveTo(p,t+i*w);
  x.lineTo(p+z,t+i*w);
  x.stroke()
 }
}

async function prep(){
 draw();

 let b=await new Promise(r=>
  cv.toBlob(r,"image/png")
 );

 if(b)
  file=new File(
   [b],
   "sudoku_result.png",
   {type:"image/png"}
  );

 shareMobile.hidden=
  !(file&&navigator.share&&navigator.canShare?.({files:[file]}))
}

async function finish(){
 if(a.some(r=>r.includes(0))){
  status.textContent="未入力のマスがあります。";
  return
 }

 if(conflicts().size){
  status.textContent="誤りがあります。";
  return
 }

 tick();
 done=true;
 sel=null;
 render();

 status.classList.add("ok");
 status.textContent=
  "正解です！ クリアタイム: "+timer.textContent;

 check.hidden=true;

 await prep();
 share.style.display="block"
}

const text=()=>
 `数独をクリアしました！\nクリアタイム: ${timer.textContent}`;

shareMobile.onclick=async()=>{
 try{
  await navigator.share({
   title:"数独クリア結果",
   text:text(),
   files:[file]
  })
 }catch(e){
  if(e.name!="AbortError")
   alert("共有できませんでした。")
 }
};

$("line").onclick=()=>
 window.open(
  `https://line.me/R/msg/text/?${encodeURIComponent(text())}`,
  "_blank"
 );

$("copy").onclick=async()=>{
 let ok=false;

 try{
  await navigator.clipboard.writeText(text());
  ok=true
 }catch{}

 if(!ok){
  let q=document.createElement("textarea");
  q.value=text();
  q.style.cssText="position:fixed;opacity:0";

  document.body.append(q);
  q.select();

  try{
   ok=document.execCommand("copy")
  }catch{}

  q.remove()
 }

 alert(ok?
  "結果文をコピーしました":
  "コピーできませんでした."
 )
};

$("save").onclick=()=>{
 let q=document.createElement("a");
 q.download="sudoku_result.png";
 q.href=cv.toDataURL("image/png");
 q.click()
};

document.querySelectorAll(".num").forEach(
 b=>b.onclick=()=>input(+b.dataset.n)
);

document.addEventListener("keydown",e=>{
 if(!sel||done)return;

 if(/^[1-9]$/.test(e.key))
  input(+e.key);
 else if(["0","Backspace","Delete"].includes(e.key)){
  e.preventDefault();
  input(0)
 }
});

check.onclick=finish;

init();
tick();
setInterval(tick,1000);
</script>
</body>
</html>