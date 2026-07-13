<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
    <meta name="theme-color" content="#050816">
    <title>Birthday Universe</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;500;700;800&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div id="app">
        <div id="loading-screen">
            <div class="loading-logo">
                <div class="planet"></div>
                <h1>Birthday Universe</h1>
                <p>Preparing your journey...</p>
                <div class="progress">
                    <div class="progress-bar"></div>
                </div>
            </div>
        </div>
        <canvas id="galaxy"></canvas>
        <div id="aurora"></div>
        <main id="scene-container"></main>
        <div id="overlay"></div>
    </div>
    <script type="module" src="script.js"></script>
  import App from "./modules/App.js";

window.addEventListener("DOMContentLoaded", () => {
    const app = new App();
    app.start();
});
export default {
    APP_NAME: "Birthday Universe",
    VERSION: "1.0",
    TARGET_FPS: 60,
    MOBILE_BREAKPOINT: 768,
    SAFE_PADDING: 18,
    COLORS: {
        background: "#050816",
        aurora1: "#5E3AFF",
        aurora2: "#1C3DFF",
        aurora3: "#892BFF"
    }
};
"use strict";

export default class Scene {
    constructor(id) {
        this.id = id;
        this.element = null;
    }

    create() {}
    enter() {}
    leave() {}
    update() {}
    destroy() {}
}
"use strict";

export default class EventBus {
    constructor() {
        this.events = new Map();
    }

    on(event, listener) {
        if (!this.events.has(event)) {
            this.events.set(event, []);
        }
        this.events.get(event).push(listener);
    }

    off(event, listener) {
        if (!this.events.has(event)) return;
        const listeners = this.events.get(event);
        const index = listeners.indexOf(listener);
        if (index > -1) {
            listeners.splice(index, 1);
        }
    }

    emit(event, payload = null) {
        if (!this.events.has(event)) return;
        for (const listener of this.events.get(event)) {
            listener(payload);
        }
    }

    clear() {
        this.events.clear();
    }
}
"use strict";

export default class StateManager {
    constructor() {
        this.state = {
            currentScene: 0,
            assetsLoaded: false,
            audioUnlocked: false,
            fireworksPlayed: false,
            memoriesViewed: 0
        };
    }

    get(key) {
        return this.state[key];
    }

    set(key, value) {
        this.state[key] = value;
    }

    update(values) {
        Object.assign(this.state, values);
    }
}
"use strict";

export default class SceneRegistry {
    constructor() {
        this.list = [];
    }

    register(scene) {
        this.list.push(scene);
    }

    get(index) {
        return this.list[index];
    }

    count() {
        return this.list.length;
    }
}
"use strict";

export default class InputManager {
    constructor() {
        this.pointer = {
            x: 0,
            y: 0
        };
        this.initialize();
    }

    initialize() {
        window.addEventListener(
            "pointermove",
            event => {
                this.pointer.x = event.clientX;
                this.pointer.y = event.clientY;
            },
            { passive: true }
        );
    }
}
"use strict";

export default class Utils {
    static clamp(value, min, max) {
        return Math.min(Math.max(value, min), max);
    }

    static lerp(a, b, t) {
        return a + (b - a) * t;
    }

    static random(min, max) {
        return Math.random() * (max - min) + min;
    }

    static randomInt(min, max) {
        return Math.floor(Utils.random(min, max + 1));
    }

    static debounce(callback, delay) {
        let timeout;
        return (...args) => {
            clearTimeout(timeout);
            timeout = setTimeout(() => {
                callback(...args);
            }, delay);
        };
    }
}
"use strict";

export default class Performance {
    constructor() {
        this.lastFrameTime = 0;
        this.fps = 60;
    }

    begin() {
        let last = performance.now();

        const tick = now => {
            const delta = now - last;
            last = now;
            this.fps = Math.round(1000 / delta);
            requestAnimationFrame(tick);
        };

        requestAnimationFrame(tick);
    }

    getFPS() {
        return this.fps;
    }
}
"use strict";

export default class TransitionController {
    constructor(container) {
        this.container = container;
        this.duration = 1200;
        this.busy = false;
    }

    async fadeOut(scene) {
        if (!scene) return;
        this.busy = true;
        scene.element.classList.remove("active");
        await this.wait();
    }

    async fadeIn(scene) {
        if (!scene) return;
        scene.element.classList.add("active");
        await this.wait();
        this.busy = false;
    }

    wait() {
        return new Promise(resolve => {
            setTimeout(resolve, this.duration);
        });
    }
}
"use strict";

export default class ParticleEngine {
    constructor(canvas) {
        this.canvas = canvas;
        this.ctx = canvas.getContext("2d");
        this.particles = [];
    }

    emit(x, y, count, config = {}) {
        const {
            type = "circle",
            color = "rgba(255, 255, 255, 0.8)",
            size = 2,
            speed = 2,
            spread = Math.PI * 2,
            speedVariance = 0.5,
            life = 1000,
            ay = 0,
            ax = 0
        } = config;

        for (let i = 0; i < count; i++) {
            const angle = Math.random() * spread;
            const particleSpeed = speed + (Math.random() - 0.5) * speedVariance;

            this.particles.push({
                x,
                y,
                vx: Math.cos(angle) * particleSpeed,
                vy: Math.sin(angle) * particleSpeed,
                ax,
                ay,
                type,
                color,
                size,
                life,
                maxLife: life,
                opacity: 1
            });
        }
    }

    update(deltaTime) {
        const dt = deltaTime / 1000;

        this.particles = this.particles.filter(p => {
            p.life -= deltaTime;

            if (p.life <= 0) {
                return false;
            }

            p.vx += p.ax * dt;
            p.vy += p.ay * dt;

            p.x += p.vx * dt;
            p.y += p.vy * dt;

            p.opacity = Math.max(0, p.life / p.maxLife);

            return true;
        });
    }

    render() {
        this.particles.forEach(p => {
            this.ctx.save();

            if (p.type === "circle") {
                this._renderCircle(p);
            } else if (p.type === "star") {
                this._renderStar(p);
            }

            this.ctx.restore();
        });
    }

    _renderCircle(p) {
        this.ctx.fillStyle = p.color.replace("1)", `${p.opacity})`);
        this.ctx.beginPath();
        this.ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
        this.ctx.fill();

        if (p.opacity > 0.3) {
            this.ctx.shadowColor = p.color.replace("1)", `${p.opacity * 0.5})`);
            this.ctx.shadowBlur = p.size * 3;
            this.ctx.shadowOffsetX = 0;
            this.ctx.shadowOffsetY = 0;

            this.ctx.beginPath();
            this.ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
            this.ctx.stroke();
        }

        this.ctx.shadowColor = "transparent";
    }

    _renderStar(p) {
        this.ctx.translate(p.x, p.y);
        this.ctx.fillStyle = p.color.replace("1)", `${p.opacity})`);

        const points = 5;
        const outerRadius = p.size;
        const innerRadius = p.size * 0.4;

        this.ctx.beginPath();
        for (let i = 0; i < points * 2; i++) {
            const radius = i % 2 === 0 ? outerRadius : innerRadius;
            const angle = (i * Math.PI) / points - Math.PI / 2;

            const x = Math.cos(angle) * radius;
            const y = Math.sin(angle) * radius;

            if (i === 0) {
                this.ctx.moveTo(x, y);
            } else {
                this.ctx.lineTo(x, y);
            }
        }

        this.ctx.closePath();
        this.ctx.fill();
    }

    clear() {
        this.particles = [];
    }
}
"use strict";

export default class AudioEngine {
    constructor() {
        this.initialized = false;
        this.audioContext = null;
        this.masterGain = null;
    }

    initialize() {
        if (this.initialized) return;

        try {
            const AudioContext = window.AudioContext || window.webkitAudioContext;
            this.audioContext = new AudioContext();
            this.masterGain = this.audioContext.createGain();
            this.masterGain.connect(this.audioContext.destination);
            this.masterGain.gain.value = 0.3;
            this.initialized = true;
        } catch (e) {
            console.warn("Web Audio API not supported:", e);
        }
    }

    resume() {
        if (this.audioContext && this.audioContext.state === "suspended") {
            this.audioContext.resume();
        }
    }

    _playTone(frequency, duration, type = "sine") {
        if (!this.audioContext) return;

        const oscillator = this.audioContext.createOscillator();
        const envelope = this.audioContext.createGain();

        oscillator.type = type;
        oscillator.frequency.value = frequency;

        envelope.gain.setValueAtTime(0.3, this.audioContext.currentTime);
        envelope.gain.exponentialRampToValueAtTime(0.01, this.audioContext.currentTime + duration);

        oscillator.connect(envelope);
        envelope.connect(this.masterGain);

        oscillator.start(this.audioContext.currentTime);
        oscillator.stop(this.audioContext.currentTime + duration);
    }

    sparkle() {
        this._playTone(800, 0.1, "sine");
        setTimeout(() => this._playTone(1200, 0.1, "sine"), 50);
    }

    chime() {
        this._playTone(523.25, 0.3, "sine");
        setTimeout(() => this._playTone(659.25, 0.3, "sine"), 150);
    }

    pop() {
        this._playTone(150, 0.1, "square");
    }

    whoosh() {
        this._playTone(200, 0.2, "triangle");
        setTimeout(() => this._playTone(100, 0.1, "triangle"), 100);
    }

    success() {
        this._playTone(523.25, 0.15, "sine");
        setTimeout(() => this._playTone(659.25, 0.15, "sine"), 100);
        setTimeout(() => this._playTone(783.99, 0.2, "sine"), 200);
    }

    typewriter() {
        this._playTone(400, 0.05, "square");
    }
}
"use strict";

export default class AssetManager {
    constructor() {
        this.assets = new Map();
        this.loaded = false;
        this.progress = 0;
    }

    async load() {
        return new Promise(resolve => {
            setTimeout(() => {
                this.loaded = true;
                this.progress = 100;
                resolve();
            }, 800);
        });
    }

    getAsset(key) {
        return this.assets.get(key);
    }

    addAsset(key, asset) {
        this.assets.set(key, asset);
    }
}
"use strict";

export default class DeviceMotionManager {
    constructor() {
        this.hasMotion = false;
        this.alpha = 0;
        this.beta = 0;
        this.gamma = 0;
        this.initialized = false;
    }

    async initialize() {
        if (!window.DeviceOrientationEvent) {
            console.warn("Device Orientation not supported");
            return;
        }

        if (typeof DeviceOrientationEvent !== "undefined" && typeof DeviceOrientationEvent.requestPermission === "function") {
            try {
                const permission = await DeviceOrientationEvent.requestPermission();
                if (permission === "granted") {
                    this._setupListeners();
                }
            } catch (error) {
                console.warn("Device Motion permission denied:", error);
            }
        } else {
            this._setupListeners();
        }

        this.initialized = true;
    }

    _setupListeners() {
        window.addEventListener("deviceorientation", e => {
            this.alpha = e.alpha || 0;
            this.beta = e.beta || 0;
            this.gamma = e.gamma || 0;
            this.hasMotion = true;
        });
    }

    getTilt() {
        return { alpha: this.alpha, beta: this.beta, gamma: this.gamma };
    }

    hasSupport() {
        return this.hasMotion;
    }
}
"use strict";

export default class AnimationManager {
    constructor() {
        this.animations = [];
        this.paused = false;
    }

    initialize() {
        document.body.classList.add("ready");
    }

    add(animation) {
        this.animations.push(animation);
        return animation;
    }

    remove(animation) {
        const index = this.animations.indexOf(animation);
        if (index > -1) {
            this.animations.splice(index, 1);
        }
    }

    clear() {
        this.animations = [];
    }

    async animateElement(element, animationName, duration = 1000) {
        return new Promise(resolve => {
            element.classList.add(animationName);
            const handleAnimationEnd = () => {
                element.classList.remove(animationName);
                element.removeEventListener("animationend", handleAnimationEnd);
                resolve();
            };
            element.addEventListener("animationend", handleAnimationEnd);
        });
    }
}
"use strict";

import TransitionController from "./TransitionController.js";
import SceneRegistry from "./SceneRegistry.js";

export default class SceneManager {
    constructor() {
        this.container = document.getElementById("scene-container");
        this.registry = new SceneRegistry();
        this.current = null;
        this.currentIndex = 0;
        this.transition = new TransitionController(this.container);
    }

    register(scene) {
        scene.create();
        this.registry.register(scene);
        this.container.appendChild(scene.element);
    }

    async start(index = 0) {
        const scene = this.registry.get(index);
        if (!scene) {
            console.error("Scene not found at index:", index);
            return;
        }

        this.current = scene;
        this.currentIndex = index;

        await this.transition.fadeIn(scene);
        scene.enter();

        console.log(`📍 Scene ${index}: ${scene.id}`);
    }

    async next() {
        if (this.transition.busy) return;

        const nextIndex = this.currentIndex + 1;
        const nextScene = this.registry.get(nextIndex);

        if (!nextScene) {
            console.log("🎉 Journey Complete!");
            return;
        }

        if (this.current) {
            this.current.leave();
            await this.transition.fadeOut(this.current);
        }

        this.current = nextScene;
        this.currentIndex = nextIndex;

        await this.transition.fadeIn(nextScene);
        nextScene.enter();

        console.log(`📍 Scene ${nextIndex}: ${nextScene.id}`);
    }

    getCurrentScene() {
        return this.current;
    }

    getTotalScenes() {
        return this.registry.count();
    }
}
"use strict";

import SceneManager from "./SceneManager.js";
import AssetManager from "./AssetManager.js";
import AudioEngine from "./AudioEngine.js";
import AnimationManager from "./AnimationManager.js";
import Performance from "./Performance.js";
import EventBus from "./EventBus.js";
import StateManager from "./StateManager.js";
import SceneRegistry from "./SceneRegistry.js";
import InputManager from "./InputManager.js";
import DeviceMotionManager from "./DeviceMotionManager.js";

import GalaxyScene from "./scenes/GalaxyScene.js";
import AuroraValleyScene from "./scenes/AuroraValleyScene.js";
import JourneyBeginsScene from "./scenes/JourneyBeginsScene.js";
import CherryBlossomScene from "./scenes/CherryBlossomScene.js";
import MemoryGalleryScene from "./scenes/MemoryGalleryScene.js";
import EnvelopeScene from "./scenes/EnvelopeScene.js";
import CakeScene from "./scenes/CakeScene.js";
import SkyLanternScene from "./scenes/SkyLanternScene.js";
import WishTreeScene from "./scenes/WishTreeScene.js";
import FireworksScene from "./scenes/FireworksScene.js";
import GalaxyFinaleScene from "./scenes/GalaxyFinaleScene.js";
import FinalMessageScene from "./scenes/FinalMessageScene.js";

export default class App {
    constructor() {
        this.loader = new AssetManager();
        this.scene = new SceneManager();
        this.animation = new AnimationManager();
        this.performance = new Performance();
        this.events = new EventBus();
        this.state = new StateManager();
        this.registry = new SceneRegistry();
        this.input = new InputManager();
        this.audio = new AudioEngine();
        this.motion = new DeviceMotionManager();

        window.app = this;

        this._registerScenes();
        this._setupEventListeners();
    }

    _registerScenes() {
        const scenes = [
            new GalaxyScene(),
            new AuroraValleyScene(),
            new JourneyBeginsScene(),
            new CherryBlossomScene(),
            new MemoryGalleryScene(),
            new EnvelopeScene(),
            new CakeScene(),
            new SkyLanternScene(),
            new WishTreeScene(),
            new FireworksScene(),
            new GalaxyFinaleScene("HAPPY BIRTHDAY"),
            new FinalMessageScene()
        ];

        scenes.forEach(scene => this.scene.register(scene));
    }

    _setupEventListeners() {
        document.addEventListener("journey:start", () => {
            console.log("🚀 Birthday Universe Journey Started!");
            this.audio.initialize();
            this.audio.resume();
            this.audio.sparkle();
            this.scene.next();
        });

        document.addEventListener("journey:next", () => {
            console.log("➡️ Next Scene");
            this.audio.chime();
            this.scene.next();
        });

        document.addEventListener(
            "click",
            () => {
                if (!this.audio.initialized) {
                    this.audio.initialize();
                    this.audio.resume();
                }
            },
            { once: true }
        );
    }

    async start() {
        this.performance.begin();
        await this.motion.initialize();
        await this.scene.start();
        this._hideLoader();
        console.log("✨ Birthday Universe - 12 Scenes Complete!");
    }

    _hideLoader() {
        const loader = document.getElementById("loading-screen");
        loader.classList.add("hidden");
    }
}
</body>
</html>
