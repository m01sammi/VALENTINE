<template>
  <div class="app" @mousemove="handleParallax">
    <div
      v-for="heart in hearts"
      :key="heart.id"
      class="heart"
      :style="heart.style"
      @click="openModal"
    ></div>

    <div v-if="isOpen" class="modal-overlay" @click.self="closeModal">
      <div class="modal">
        <h2 class="title neon">💖 С Днём Святого Валентина 💖</h2>

        <p class="message">
          Любимая, ты — самое прекрасное, что случилось в моей жизни. Каждый
          день рядом с тобой наполнен теплом, счастьем и любовью. Я благодарен
          судьбе за тебя и люблю бесконечно 💕
        </p>

        <div class="coupon-wrapper">
          <div
            class="coupon"
            :class="{ revealed: isCouponRevealed }"
            @mousemove="tilt"
            @mouseleave="resetTilt"
            ref="couponRef"
          >
            <div class="coupon-content">
              <h3 v-if="!isCouponRevealed">🎟 Купон любви</h3>

              <div v-else class="revealed-content">
                <h2 class="big-text neon">
                  Купон на исполнение любого желания
                </h2>
                <div v-if="couponCount === 1" class="counter">Использован</div>
              </div>
            </div>

            <div
              v-if="!isCouponRevealed"
              class="tear-off"
              @click="revealCoupon"
            >
              ОТОРВАТЬ
            </div>
          </div>
        </div>

        <button @click="closeModal">Закрыть</button>
      </div>
    </div>

    <div
      v-for="conf in confetti"
      :key="conf.id"
      class="confetti"
      :style="conf.style"
    >
      💖
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const hearts = ref([]);
const confetti = ref([]);
const isOpen = ref(false);
const isCouponRevealed = ref(false);
const couponRef = ref(null);
const couponCount = ref(0);

const createHearts = () => {
  for (let i = 0; i < 40; i++) {
    hearts.value.push({
      id: i,
      style: {
        left: Math.random() * 100 + "vw",
        top: "100vh",
        animationDuration: 8 + Math.random() * 6 + "s",
        animationDelay: Math.random() * 5 + "s",
        transform: `scale(${0.5 + Math.random()})`,
      },
    });
  }
};

const openModal = () => {
  isOpen.value = true;
};

const closeModal = () => {
  isOpen.value = false;
  isCouponRevealed.value = false;
};

const playTearSound = () => {
  const ctx = new (window.AudioContext || window.webkitAudioContext)();
  const buffer = ctx.createBuffer(1, ctx.sampleRate * 0.2, ctx.sampleRate);
  const data = buffer.getChannelData(0);

  for (let i = 0; i < data.length; i++) {
    data[i] = Math.random() * 2 - 1;
  }

  const source = ctx.createBufferSource();
  source.buffer = buffer;

  const gain = ctx.createGain();
  gain.gain.setValueAtTime(0.3, ctx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.2);

  source.connect(gain);
  gain.connect(ctx.destination);
  source.start();
};

const launchConfetti = () => {
  for (let i = 0; i < 25; i++) {
    confetti.value.push({
      id: Date.now() + i,
      style: {
        left: Math.random() * 100 + "vw",
        animationDuration: 2 + Math.random() * 2 + "s",
      },
    });
  }

  setTimeout(() => {
    confetti.value = [];
  }, 3000);
};

const revealCoupon = () => {
  playTearSound();
  isCouponRevealed.value = true;
  launchConfetti();
};

const tilt = (e) => {
  const rect = couponRef.value.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;

  const rotateX = -(y - rect.height / 2) / 20;
  const rotateY = (x - rect.width / 2) / 20;

  couponRef.value.style.transform = `rotateX(${rotateX}deg) rotateY(${rotateY}deg)`;
};

const resetTilt = () => {
  couponRef.value.style.transform = "rotateX(0) rotateY(0)";
};

const handleParallax = (e) => {
  const x = (window.innerWidth / 2 - e.clientX) / 40;
  const y = (window.innerHeight / 2 - e.clientY) / 40;

  document.documentElement.style.setProperty("--parallaxX", `${x}px`);
  document.documentElement.style.setProperty("--parallaxY", `${y}px`);
};

const fetchCount = async () => {
  await fetch("https://698ecc96aded595c2532d5be.mockapi.io/valentine")
    .then((res) => res.json())
    .then((res) => {
      couponCount.value = res[0]?.count ?? 0;
    });
};

onMounted(() => {
  createHearts();
  fetchCount();
});
</script>

<style>
/* ================= BASE ================= */

button {
  margin-top: 20px;
  padding: 12px 24px;
  border-radius: 50px;
  border: none;
  background: linear-gradient(135deg, #ff4d6d, #ff758f);
  color: white;
  font-weight: 600;
  letter-spacing: 0.5px;
  cursor: pointer;
  transition: all 0.1s ease;
}

button:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(255, 77, 109, 0.4);
}

.neon {
  color: #fff;
  animation: neonBreath 3s ease-in-out infinite;
}

/* ================= LAYOUT ================= */

.app {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  color: white;

  --parallaxX: 0px;
  --parallaxY: 0px;

  background:
    radial-gradient(
      circle at 25% 25%,
      rgba(255, 170, 150, 0.25),
      transparent 45%
    ),
    radial-gradient(
      circle at 75% 75%,
      rgba(255, 120, 140, 0.25),
      transparent 45%
    ),
    radial-gradient(circle at center, #2a0f18, #14070d 60%, #0d0408);

  transform: translate(var(--parallaxX), var(--parallaxY));
  transition: transform 0.1s ease-out;
}

/* ================= HEARTS ================= */

.heart {
  position: absolute;
  width: 20px;
  height: 20px;
  background: #ff4d6d;
  transform: rotate(45deg);
  animation: float linear infinite;
  cursor: pointer;
}

.heart::before,
.heart::after {
  content: "";
  position: absolute;
  width: 20px;
  height: 20px;
  background: #ff4d6d;
  border-radius: 50%;
}

.heart::before {
  top: -10px;
}
.heart::after {
  left: -10px;
}

/* ================= MODAL ================= */

.modal-overlay {
  position: fixed;
  inset: 0;
  backdrop-filter: blur(12px);
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  animation: fadeOverlay 0.4s ease;
}

.modal {
  width: 520px;
  max-width: 92vw;

  padding: 50px;
  border-radius: 30px;

  background: rgba(255, 230, 235, 0.06);
  backdrop-filter: blur(30px);

  border: 1px solid rgba(255, 200, 210, 0.25);

  box-shadow:
    0 30px 80px rgba(0, 0, 0, 0.6),
    inset 0 0 60px rgba(255, 180, 190, 0.05);

  text-align: center;
  animation: modalIn 0.5s cubic-bezier(0.2, 0.8, 0.2, 1);
}

/* ================= COUPON ================= */

.coupon-wrapper {
  display: flex;
  justify-content: center;
  margin: 30px 0;
}

.coupon {
  width: 360px;
  padding: 30px;
  border-radius: 24px;
  background: rgba(255, 255, 255, 0.06);
  backdrop-filter: blur(25px);
  border: 1px solid rgba(255, 255, 255, 0.15);
  box-shadow:
    0 15px 50px rgba(0, 0, 0, 0.5),
    inset 0 0 40px rgba(255, 255, 255, 0.05);
  position: relative;
  overflow: hidden;
  transition: all 0.1s ease;
  animation: couponFloat 5s ease-in-out infinite;
}

.coupon:hover {
  transform: translateY(-5px);
  box-shadow:
    0 25px 60px rgba(255, 77, 109, 0.4),
    inset 0 0 60px rgba(255, 255, 255, 0.08);
}

.tear-off {
  position: absolute;
  right: 0;
  top: 0;
  width: 90px;
  height: 100%;
  background: linear-gradient(180deg, #ff4d6d, #ff758f);
  border-left: 2px dashed rgba(255, 255, 255, 0.4);
  writing-mode: vertical-rl;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  cursor: pointer;
  clip-path: polygon(
    0 0,
    100% 0,
    100% 100%,
    0% 95%,
    10% 90%,
    0% 85%,
    10% 80%,
    0% 75%,
    10% 70%,
    0% 65%,
    10% 60%,
    0% 55%,
    10% 50%,
    0% 45%,
    10% 40%,
    0% 35%,
    10% 30%,
    0% 25%,
    10% 20%,
    0% 15%
  );
  transition:
    transform 0.7s ease,
    opacity 0.7s ease;
}

.coupon.revealed .tear-off {
  transform: translateX(120%) rotate(25deg);
  opacity: 0;
}

/* ================= CONFETTI ================= */

.confetti {
  position: fixed;
  top: -20px;
  font-size: 20px;
  animation: confettiFall linear forwards;
}

/* ================= ANIMATIONS ================= */

@keyframes float {
  0% {
    transform: translateY(0) rotate(45deg);
    opacity: 0;
  }
  10% {
    opacity: 1;
  }
  50% {
    transform: translateY(-50vh) translateX(20px) rotate(45deg);
  }
  100% {
    transform: translateY(-120vh) translateX(-20px) rotate(45deg);
    opacity: 0;
  }
}

@keyframes modalIn {
  from {
    opacity: 0;
    transform: translateY(40px) scale(0.95);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

@keyframes fadeOverlay {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes confettiFall {
  to {
    transform: translateY(100vh) rotate(720deg);
    opacity: 0;
  }
}

@keyframes couponFloat {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-6px);
  }
}

@media (max-width: 768px) {
  .modal {
    width: 100%;
    max-width: 100%;
    height: 100vh;
    border-radius: 0;
    padding: 40px 24px;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }

  .message {
    font-size: 15px;
    line-height: 1.6;
  }

  .coupon {
    width: 100%;
    max-width: 320px;
    padding: 24px;
  }

  .tear-off {
    width: 70px;
    font-size: 12px;
  }

  button {
    width: 100%;
  }
}

@keyframes neonBreath {
  0%,
  100% {
    text-shadow:
      0 0 6px rgba(255, 120, 140, 0.6),
      0 0 14px rgba(255, 150, 170, 0.5);
  }
  50% {
    text-shadow:
      0 0 10px rgba(255, 150, 170, 0.8),
      0 0 24px rgba(255, 120, 140, 0.6);
  }
}
</style>
