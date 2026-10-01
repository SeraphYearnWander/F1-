# F1-<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover, user-scalable=no">
<title>F1 极速围场</title>
<style>
  :root{
    --navy:#12202F; --navy2:#1D3047; --navy3:#2A3C52;
    --yellow:#FFD400; --purple:#B94BF2; --red:#E10600; --green:#2FD07B;
    --concrete:#C9D0D4;
    --reserve:0px;
  }
  *{box-sizing:border-box}
  html,body{margin:0;height:100%}
  body{
    background:var(--concrete);
    color:#fff;
    font-family:"Bahnschrift","DIN Alternate","Arial Narrow","PingFang SC","Microsoft YaHei","Noto Sans SC",sans-serif;
    display:grid;place-items:center;min-height:100dvh;
    overscroll-behavior:none;user-select:none;-webkit-user-select:none;
  }
  .game{display:grid;gap:10px;justify-items:center}
  .stage{
    position:relative;
    width:min(calc(100vw - 16px), calc((100dvh - var(--reserve) - 16px) * .6667), 520px);
    aspect-ratio:2/3;
    container-type:inline-size;
    border-radius:6px;overflow:hidden;background:#48853A;
    box-shadow:0 0 0 5px var(--navy), 0 16px 40px rgba(18,32,47,.35);
    touch-action:none;
  }
  canvas{display:block;width:100%;height:100%}

  /* HUD：仿转播计时塔 */
  .hud{position:absolute;inset:2.4cqw 2.4cqw auto;display:grid;grid-template-columns:1fr auto 1fr;gap:2cqw;align-items:start;pointer-events:none}
  .tile{background:rgba(18,32,47,.92);padding:1.4cqw 2.8cqw 1.6cqw;border-radius:3px;border-bottom:.7cqw solid var(--yellow);line-height:1.1}
  .tile b{display:block;font-size:7.4cqw;font-style:italic;font-weight:700;font-variant-numeric:tabular-nums}
  .tile span{display:block;font-size:2.8cqw;opacity:.8}
  .tile.speed{justify-self:start;min-width:21cqw}
  .tile.lap{text-align:center;min-width:25cqw}
  .tile.score{justify-self:end;text-align:right;min-width:21cqw}
  .tile.pb{border-bottom-color:var(--purple)}

  .ers{
    position:absolute;left:2.4cqw;right:21cqw;bottom:2.4cqw;
    display:flex;align-items:center;gap:2cqw;
    background:rgba(18,32,47,.92);padding:1.6cqw 2.6cqw;border-radius:3px;
    font-size:3cqw;pointer-events:none;
  }
  .ers .bar{flex:1;height:2.2cqw;background:var(--navy3);overflow:hidden}
  .ers i{display:block;height:100%;width:50%;background:var(--yellow);transition:width .1s linear}
  .ers.low i{background:var(--red)}
  .ers.on i{background:var(--green)}
  .ers em{font-style:normal;min-width:12cqw;text-align:right;opacity:.9}

  .mute{
    position:absolute;right:2.4cqw;bottom:2.4cqw;width:17cqw;
    font:inherit;font-size:3cqw;color:#fff;background:rgba(18,32,47,.92);
    border:0;border-radius:3px;padding:1.9cqw 0;cursor:pointer;
  }
  .mute:focus-visible,.go:focus-visible,.pad button:focus-visible{outline:.6cqw solid #fff;outline-offset:.5cqw}

  .flash{
    position:absolute;left:50%;top:25%;transform:translateX(-50%);
    background:var(--navy);padding:1.8cqw 4cqw;border-left:1.2cqw solid var(--yellow);
    font-size:4.6cqw;font-style:italic;font-weight:700;white-space:nowrap;
    opacity:0;transition:opacity .2s;pointer-events:none;
  }
  .flash.show{opacity:1}
  .flash.purple{border-left-color:var(--purple)}

  /* 菜单 / 结算 */
  .overlay{
    position:absolute;inset:0;display:grid;align-content:start;
    padding:8cqw 5cqw 5cqw;
    background:linear-gradient(to bottom, rgba(18,32,47,.95) 58%, rgba(18,32,47,0));
  }
  .overlay[hidden]{display:none}
  .panel{border-left:1.2cqw solid var(--yellow);padding-left:4cqw;display:grid;gap:3.2cqw;justify-items:start}
  .panel h1{margin:0;font-size:10.5cqw;line-height:1;font-style:italic;font-weight:800}
  .lead{margin:0;font-size:3.9cqw;line-height:1.55;max-width:32em;opacity:.92}
  .keys{display:grid;grid-template-columns:1fr 1fr;gap:1.8cqw 3cqw;font-size:3.2cqw;margin:0;padding:0;list-style:none;align-items:center}
  kbd{background:var(--navy3);border-bottom:.5cqw solid #0B1520;border-radius:2px;padding:.3cqw 1.5cqw;font:inherit;margin-right:.8cqw}
  .best{display:flex;gap:5cqw;margin:0;font-size:3.4cqw;opacity:.95}
  .best b{color:var(--purple);font-style:italic;font-size:4.4cqw}
  .stats{display:grid;grid-template-columns:repeat(3,auto);gap:3cqw 6cqw}
  .stats b{display:block;font-size:7cqw;font-style:italic;font-variant-numeric:tabular-nums;line-height:1.1}
  .stats b.pb{color:var(--purple)}
  .stats span{font-size:3cqw;opacity:.8}
  .go{
    font:inherit;font-size:4.8cqw;font-weight:700;background:var(--yellow);color:var(--navy);
    border:0;border-radius:3px;padding:2.2cqw 8cqw;cursor:pointer;
  }
  .go:hover{filter:brightness(1.08)}
  .go:active{transform:translateY(1px)}

  /* 触屏按键 */
  .pad{display:none;width:min(calc(100vw - 16px),520px);justify-content:space-between;gap:10px}
  .pad .grp{display:flex;gap:10px}
  .pad button{
    font:inherit;font-size:18px;font-weight:700;color:#fff;background:var(--navy);
    border:0;border-bottom:4px solid #0B1520;border-radius:6px;min-width:64px;height:64px;padding:0 16px;
    touch-action:none;-webkit-tap-highlight-color:transparent;
  }
  .pad button.on{back
