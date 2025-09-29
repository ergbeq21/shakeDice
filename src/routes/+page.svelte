<!-- +page.svelte -->
<script>
  import { onMount, onDestroy } from 'svelte';
  import { browser } from '$app/environment';

  let value = 1, listening = false, lastRoll = 0;
  let trackX = true, trackY = true;

  let ax=0, ay=0, az=0, gx=0, gy=0, gz=0, alpha=0, beta=0, gamma=0;

  const THRESHOLD = 13, COOLDOWN_MS = 800;
  const pips = {1:[4],2:[0,8],3:[0,4,8],4:[0,2,6,8],5:[0,2,4,6,8],6:[0,2,3,5,6,8]};
  const roll = () => (value = 1 + Math.floor(Math.random()*6));

  const clamp = (n, min, max) => Math.max(min, Math.min(max, n));
  function dieTransform() {
    const tx = trackY ? clamp(gx * 2, -14, 14) : 0;
    const ty = trackX ? clamp(-gy * 2, -14, 14) : 0;
    const rz = clamp(gamma * 0.6, -20, 20);
    return `translate3d(${tx}px, ${ty}px, 0) rotate(${rz}deg)`;
  }

  let sAx=0,sAy=0,sAz=0,sGx=0,sGy=0,sGz=0,sGamma=0;
  const SMOOTH=0.25;
  const smooth = (prev,next)=> prev+(next-prev)*SMOOTH;

  function handleMotion(e) {
    const a = e.acceleration || {}, ag = e.accelerationIncludingGravity || {};
    ax = sAx = smooth(sAx,a.x??0);
    ay = sAy = smooth(sAy,a.y??0);
    az = sAz = smooth(sAz,a.z??0);
    gx = sGx = smooth(sGx,ag.x??0);
    gy = sGy = smooth(sGy,ag.y??0);
    gz = sGz = smooth(sGz,ag.z??0);

    const now = Date.now();
    const shakeX = trackX && Math.abs(ax) > THRESHOLD;
    const shakeY = trackY && Math.abs(ay) > THRESHOLD;
    if ((shakeX || shakeY) && now - lastRoll > COOLDOWN_MS) {
      lastRoll = now; roll();
      spinning = true; clearTimeout(spinT); spinT=setTimeout(()=>spinning=false,500);
    }
  }
  function handleOrientation(e){ alpha=e.alpha??0; beta=e.beta??0; gamma=sGamma=smooth(sGamma,e.gamma??0); }

  let spinning=false, spinT;
  async function start() {
    if(!browser) return;
    if(typeof DeviceMotionEvent!=='undefined' && DeviceMotionEvent.requestPermission){
      const p=await DeviceMotionEvent.requestPermission(); if(p!=='granted') return;
    }
    window.addEventListener('devicemotion',handleMotion,{passive:true});
    window.addEventListener('deviceorientation',handleOrientation,{passive:true});
    listening=true;
  }
  function stop(){ if(!browser) return; window.removeEventListener('devicemotion',handleMotion); window.removeEventListener('deviceorientation',handleOrientation); listening=false; }

  onMount(start); onDestroy(stop);
</script>

<div class="min-h-screen bg-gradient-to-br from-pink-200 via-purple-200 to-cyan-200 flex items-center justify-center p-6 font-sans text-[16px] text-purple-800">
  <div class="space-y-8 w-full max-w-sm">

    <!-- Buttons -->
    <div class="flex items-center gap-6 justify-center">
      {#if !listening}
        <button 
          onclick={start} 
          class="px-4 py-2 rounded-full bg-gradient-to-r from-yellow-400 to-pink-400 hover:scale-105 transform transition 
                 shadow-lg shadow-pink-400/40 font-bold text-purple-900">
          Activate
        </button>
      {:else}
        <button 
          onclick={roll} 
          class="px-4 py-2 rounded-full bg-gradient-to-r from-cyan-400 to-blue-400 hover:scale-105 transform transition 
                 shadow-lg shadow-cyan-400/40 font-bold text-purple-900">
        Roll it
        </button>
      {/if}
    </div>

    <!-- Dice -->
    <div
      class={`h-44 w-44 mx-auto bg-gradient-to-br from-yellow-100 to-pink-100 border-4 border-purple-300 rounded-3xl grid grid-cols-3 gap-3 p-4 
              shadow-[0_6px_20px_rgba(0,0,0,0.15)] 
              transition-transform duration-200 ease-out
              ${spinning ? 'animate-bounce' : ''}`}
      style={`transform:${dieTransform()}`}
    >
      {#each Array(9) as _, i}
        <div class="flex items-center justify-center">
          {#if pips[value].includes(i)}
            <div class="w-6 h-6 rounded-full bg-gradient-to-br from-pink-400 to-cyan-400 shadow-[0_0_10px_2px_rgba(255,105,180,0.6)]"></div>
          {/if}
        </div>
      {/each}
    </div>

    <!-- Result -->
    <div class="text-center text-xl font-bold text-purple-700 tracking-wide">
      RESULT: <span class="text-pink-500">{value}</span> 
    </div>
  </div>
</div>


<style>
  @keyframes dice-spin {
    0%   { transform: scale(1) rotate(0deg); }
    25%  { transform: scale(1.1) rotate(90deg); }
    50%  { transform: scale(1.05) rotate(180deg); }
    75%  { transform: scale(1.08) rotate(270deg); }
    100% { transform: scale(1) rotate(360deg); }
  }
  .animate-dice-spin {
    animation: dice-spin 0.5s cubic-bezier(.4,.6,.6,1.2);
  }
</style>
