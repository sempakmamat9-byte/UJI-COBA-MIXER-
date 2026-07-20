
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>DJ Mixer FADER FIX</title>
<style>
    * {margin: 0; padding: 0; box-sizing: border-box; font-family: Arial Black; user-select: none; touch-action: none;}
    body {background: #0a0a0a; display: flex; justify-content: center; align-items: center; min-height: 100vh; flex-direction: column;}
.mixer {background: linear-gradient(180deg, #2a2a2a 0%, #1a1a1a 100%); border-radius: 12px; padding: 20px; display: flex; gap: 0; box-shadow: 0 10px 40px rgba(0,0,0,0.8); border: 1px solid #333; margin-top: 20px;}
.channel {padding: 0 18px; display: flex; flex-direction: column; align-items: center; border-left: 1px solid #2a2a2a;}
.master {padding: 0 18px; display: flex; flex-direction: column; align-items: center; border-left: 2px solid #333;}
.label {color: #ddd; font-size: 12px; font-weight: bold; margin-bottom: 8px; text-transform: uppercase;}
.knob-box {display: flex; align-items: center; gap: 5px; margin: 5px 0;}
.knob {width: 40px; height: 40px; border-radius: 50%; background: radial-gradient(circle at 30% 30%, #666, #222); border: 2px solid #444; position: relative; box-shadow: 0 3px 6px rgba(0,0,0,0.6); transition: transform 0.1s;}
.knob::after {content: ''; position: absolute; width: 2px; height: 14px; background: #eee; top: 4px; left: 50%; transform: translateX(-50%); border-radius: 2px;}
.btn {width: 20px; height: 20px; background: #444; color: white; border: none; cursor: pointer; border-radius: 3px;}
.btn:active {background: #666;}
.fader-wrap {margin-top: 15px; position: relative; height: 200px; width: 30px; display: flex; align-items: center; justify-content: center;}
.scale {position: absolute; left: -25px; height: 100%; display: flex; flex-direction: column; justify-content: space-between; color: #888; font-size: 10px; pointer-events: none;}
.fader-track {width: 8px; height: 100%; background: #000; border-radius: 4px; position: relative;}
.fader {width: 28px; height: 30px; background: linear-gradient(180deg, #ddd, #aaa); border-radius: 3px; position: absolute; left: 50%; transform: translateX(-50%); cursor: grab; box-shadow: 0 2px 5px rgba(0,0,0,0.5); z-index: 10;}
.fader:active {cursor: grabbing; background: linear-gradient(180deg, #fff, #ccc);}
.ch-label {margin-top: 10px; color: #eee; font-size: 16px; font-weight: bold;}
.controls {color: white; margin-bottom: 10px;}
</style>
</head>
<body>

<div class="controls">
    <input type="file" id="audioFile" accept=".mp3">
    <button id="playBtn">PLAY</button>
    <button id="stopBtn">STOP</button>
</div>

<div class="mixer">
    <div class="channel">
        <div class="label">GAIN</div>
        <div class="knob-box">
            <button class="btn" data-target="gain" data-dir="-1">-</button>
            <div class="knob" id="gainKnob"></div>
            <button class="btn" data-target="gain" data-dir="1">+</button>
        </div>

        <div class="label">HIGH</div>
        <div class="knob-box">
            <button class="btn" data-target="high" data-dir="-1">-</button>
            <div class="knob" id="highKnob"></div>
            <button class="btn" data-target="high" data-dir="1">+</button>
        </div>

        <div class="label">MID</div>
        <div class="knob-box">
            <button class="btn" data-target="mid" data-dir="-1">-</button>
            <div class="knob" id="midKnob"></div>
            <button class="btn" data-target="mid" data-dir="1">+</button>
        </div>

        <div class="label">LOW</div>
        <div class="knob-box">
            <button class="btn" data-target="low" data-dir="-1">-</button>
            <div class="knob" id="lowKnob"></div>
            <button class="btn" data-target="low" data-dir="1">+</button>
        </div>

        <div class="fader-wrap">
            <div class="scale"><span>10</span><span></span><span>5</span><span></span><span>0</span><span></span><span>-5</span><span></span><span>-10</span></div>
            <div class="fader-track"><div class="fader" id="chFader"></div></div>
        </div>
        <div class="ch-label">CH1</div>
    </div>

    <div class="master">
        <div class="label">MASTER</div>
        <div class="fader-wrap">
            <div class="scale"><span>10</span><span></span><span>5</span><span></span><span>0</span><span></span><span>-5</span><span></span><span>-10</span></div>
            <div class="fader-track"><div class="fader" id="masterFader"></div></div>
        </div>
    </div>
</div>

<script>
const AudioContext = window.AudioContext || window.webkitAudioContext;
const audioCtx = new AudioContext();

let audioSource;
let chGain, chLow, chMid, chHigh;
let masterGain;

// SETUP AUDIO
masterGain = audioCtx.createGain();
masterGain.connect(audioCtx.destination);
masterGain.gain.value = 0.7;

chGain = audioCtx.createGain();
chGain.gain.value = 0.5;

chLow = audioCtx.createBiquadFilter(); chLow.type = "lowshelf"; chLow.frequency.value = 250; chLow.gain.value = 0;
chMid = audioCtx.createBiquadFilter(); chMid.type = "peaking"; chMid.frequency.value = 1000; chMid.Q.value = 1.0; chMid.gain.value = 0;
chHigh = audioCtx.createBiquadFilter(); chHigh.type = "highshelf"; chHigh.frequency.value = 4000; chHigh.gain.value = 0;

chGain.connect(chLow).connect(chMid).connect(chHigh).connect(masterGain);

// UPLOAD MP3
document.getElementById('audioFile').addEventListener('change', async (e) => {
    const file = e.target.files[0];
    const arrayBuffer = await file.arrayBuffer();
    const audioBuffer = await audioCtx.decodeAudioData(arrayBuffer);
    if(audioSource) audioSource.stop();
    audioSource = audioCtx.createBufferSource();
    audioSource.buffer = audioBuffer;
    audioSource.loop = true;
    audioSource.connect(chGain);
});

document.getElementById('playBtn').onclick = () => { audioCtx.resume(); if(audioSource) audioSource.start(0); }
document.getElementById('stopBtn').onclick = () => { if(audioSource) audioSource.stop(); }

// KNOB
const knobValues = {gain: 0.5, low: 0, mid: 0, high: 0};
const knobAngles = {gain: 0, low: 0, mid: 0, high: 0};
function updateKnob(type){
    document.getElementById(type+'Knob').style.transform = `rotate(${knobAngles[type]}deg)`;
    if(type == 'gain') chGain.gain.value = knobValues.gain * 2;
    if(type == 'low') chLow.gain.value = knobValues.low;
    if(type == 'mid') chMid.gain.value = knobValues.mid;
    if(type == 'high') chHigh.gain.value = knobValues.high;
}
document.querySelectorAll('.btn').forEach(btn => {
    btn.addEventListener('click', () => {
        let type = btn.dataset.target; let dir = parseFloat(btn.dataset.dir);
        knobAngles[type] += dir * 20;
        knobValues[type] += dir * 1.5;
        knobValues.gain = Math.max(0, Math.min(2, knobValues.gain));
        knobValues.low = Math.max(-15, Math.min(15, knobValues.low));
        knobValues.mid = Math.max(-15, Math.min(15, knobValues.mid));
        knobValues.high = Math.max(-15, Math.min(15, knobValues.high));
        updateKnob(type);
    });
});

//========================================
// FADER FIX PAKE POINTER EVENTS
// ISO NENG HP + PC
//========================================
let activeFader = null;

function setFaderPosition(fader, y){
    const track = fader.parentElement;
    const rect = track.getBoundingClientRect();
    let posY = y - rect.top;
    posY = Math.max(0, Math.min(posY, rect.height));
    fader.style.top = posY + 'px';

    let val = 1 - (posY / rect.height); // 0 - 1

    if(fader.id == 'chFader') chGain.gain.value = val;
    if(fader.id == 'masterFader') masterGain.gain.value = val;
}

document.querySelectorAll('.fader').forEach(fader => {
    fader.addEventListener('pointerdown', (e) => {
        activeFader = fader;
        fader.setPointerCapture(e.pointerId);
    });
});

document.addEventListener('pointermove', (e) => {
    if(!activeFader) return;
    setFaderPosition(activeFader, e.clientY);
});

document.addEventListener('pointerup', () => {
    activeFader = null;
});

// SET AWAL FADER TENGAH
window.onload = () => {
    document.querySelectorAll('.fader').forEach(f => {
        const trackHeight = f.parentElement.clientHeight;
        f.style.top = (trackHeight/2 - 15) + 'px';
    });
}
</script>

</body>
</html>
