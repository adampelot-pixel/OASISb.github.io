<!DOCTYPE html>
<html lang="fr">
  <head>
    <meta charset="UTF-8" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover"
    />
    <title>OASIS BOXING BEEP</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
      body {
        background: #000;
        color: #fff;
        font-family: system-ui, sans-serif;
        height: 100vh;
        display: flex;
        flex-direction: column;
        overflow: hidden;
        margin: 0;
      }
      .timer-huge {
        font-size: 48vw;
        font-weight: 900;
        line-height: 0.8;
        font-variant-numeric: tabular-nums;
      }
      #btn-main {
        -webkit-tap-highlight-color: transparent;
      }
      .bg-work {
        background-color: #be123c !important;
      } /* Rouge */
      .bg-rest {
        background-color: #15803d !important;
      } /* Vert */
      .bg-prep {
        background-color: #000000 !important;
      } /* Noir */
    </style>
  </head>
  <body id="main-bg" class="p-6 transition-colors duration-300">
    <div class="w-full flex justify-between pt-4">
      <div
        id="round-info"
        class="text-[10px] font-black opacity-50 tracking-[0.2em] uppercase"
      >
        Cycle 1 • Round 1/4
      </div>
      <button
        onclick="document.documentElement.requestFullscreen()"
        class="text-[10px] border border-white/20 px-3 py-1 rounded-full uppercase"
      >
        Full 🔳
      </button>
    </div>

    <div
      class="flex-grow flex flex-col justify-center items-center text-center"
    >
      <p
        id="status"
        class="text-blue-500 font-black tracking-[0.5em] text-xs uppercase mb-4"
      >
        Préparation
      </p>
      <div id="timer" class="timer-huge italic">05</div>
      <p
        id="exo-name"
        class="text-2xl font-black mt-6 uppercase tracking-tight"
      >
        OASIS TRAINING
      </p>
    </div>

    <div class="pb-12 px-4">
      <button
        id="btn-main"
        onclick="toggleWorkout()"
        class="w-full bg-white text-black py-8 rounded-[40px] font-black text-3xl shadow-2xl active:scale-95 transition-all"
      >
        START
      </button>
    </div>

    <script>
      const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

      // FONCTION BIP (Petits bips et Grosse Sirène)
      function playSound(type) {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.connect(gain);
        gain.connect(audioCtx.destination);

        if (type === "tick") {
          // Petit bip (compte à rebours)
          osc.type = "sine";
          osc.frequency.setValueAtTime(600, audioCtx.currentTime);
          gain.gain.setValueAtTime(0.2, audioCtx.currentTime);
          gain.gain.exponentialRampToValueAtTime(
            0.01,
            audioCtx.currentTime + 0.1
          );
          osc.start();
          osc.stop(audioCtx.currentTime + 0.1);
        } else if (type === "horn") {
          // GROSSE SIRÈNE (Transition)
          osc.type = "sawtooth"; // Son plus agressif
          osc.frequency.setValueAtTime(150, audioCtx.currentTime);
          osc.frequency.exponentialRampToValueAtTime(
            100,
            audioCtx.currentTime + 0.6
          );
          gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
          gain.gain.linearRampToValueAtTime(0, audioCtx.currentTime + 0.6);
          osc.start();
          osc.stop(audioCtx.currentTime + 0.6);
        }
      }

      const cycles = [
        {
          n: "CARDIO",
          e: ["Jumping jacks", "Squats", "Genoux", "Jump squats"],
        },
        { n: "HAUT", e: ["Pompes", "Planche", "Épaules", "Dips"] },
        { n: "ABDOS", e: ["Crunchs", "Gainage", "Jambes", "Russian twist"] },
      ];

      let cIdx = 0,
        rIdx = 0,
        time = 5,
        active = false,
        mode = "PREP",
        timer;

      function updateUI() {
        const bg = document.getElementById("main-bg");
        const status = document.getElementById("status");
        document.getElementById("timer").innerText =
          time < 10 ? "0" + time : time;
        document.getElementById("round-info").innerText = `Cycle ${
          cIdx + 1
        }/3 • Round ${rIdx + 1}/4`;

        if (mode === "WORK") {
          bg.className = "p-6 bg-work transition-colors duration-300";
          status.innerText = "TRAVAIL";
          status.className =
            "text-white font-black tracking-[0.5em] text-xs uppercase mb-4";
          document.getElementById("exo-name").innerText = cycles[cIdx].e[rIdx];
        } else {
          bg.className =
            mode === "REPOS"
              ? "p-6 bg-rest transition-colors duration-300"
              : "p-6 bg-prep transition-colors duration-300";
          status.innerText = mode === "REPOS" ? "REPOS" : "PRÉPA";
          status.className =
            "text-blue-500 font-black tracking-[0.5em] text-xs uppercase mb-4";
          document.getElementById("exo-name").innerText =
            mode === "REPOS" ? "Récupère" : "Suivant : " + cycles[cIdx].e[rIdx];
        }
      }

      function toggleWorkout() {
        if (audioCtx.state === "suspended") audioCtx.resume();
        if (active) {
          clearInterval(timer);
          active = false;
          document.getElementById("btn-main").innerText = "CONTINUER";
          return;
        }
        active = true;
        document.getElementById("btn-main").innerText = "PAUSE";
        document.getElementById("btn-main").style.backgroundColor =
          "rgba(255,255,255,0.2)";
        document.getElementById("btn-main").style.color = "white";

        timer = setInterval(() => {
          time--;
          if (time <= 3 && time > 0) playSound("tick"); // Bips 3, 2, 1
          if (time < 0) {
            playSound("horn"); // GROS BIP DE CHANGEMENT
            logic();
          }
          updateUI();
        }, 1000);
      }

      function logic() {
        if (mode === "PREP") {
          mode = "WORK";
          time = 30;
        } else if (mode === "WORK") {
          if (rIdx < 3) {
            mode = "REPOS";
            time = 15;
          } else if (cIdx < 2) {
            mode = "REPOS";
            time = 30;
            rIdx = -1;
            cIdx++;
          } else {
            finish();
          }
        } else {
          rIdx++;
          mode = "WORK";
          time = 30;
        }
      }

      function finish() {
        clearInterval(timer);
        document.getElementById("timer").innerText = "🏆";
        document.getElementById("exo-name").innerText = "SESSION FINIE";
      }
    </script>
  </body>
</html>
