# SoleGoes Landing Page - Technical Explanation

## Overview
This landing page uses **GSAP (GreenSock Animation Platform)** with **ScrollTrigger** for scroll-based animations and **Lenis** for smooth scrolling. The main feature is a "scrollytelling" section where a phone mockup stays fixed while content scrolls, creating an interactive narrative.

---

## 1. Smooth Scrolling with Lenis

### What is Lenis?
Lenis is a lightweight smooth scroll library that creates a buttery-smooth scrolling experience by interpolating scroll positions.

### Implementation (Lines 285-297)
```javascript
const lenis = new Lenis({
    smooth: true,
    lerp: 0.1,              // Linear interpolation factor (0-1)
    wheelMultiplier: 0.3,   // Scroll speed multiplier
    touchMultiplier: 0.8,   // Touch scroll speed
    infinite: false,
});

function raf(time) { 
    lenis.raf(time); 
    requestAnimationFrame(raf); 
}
requestAnimationFrame(raf);
```

### How it Works:
1. **`lerp: 0.1`** - Controls smoothness. Lower = smoother but slower response
   - Value of 0.1 means: "Move 10% of the distance to target position each frame"
   - Creates the "heavy/slow" feel you experience

2. **`wheelMultiplier: 0.3`** - Reduces scroll speed to 30% of normal
   - Makes scrolling feel more deliberate and cinematic

3. **`requestAnimationFrame(raf)`** - Updates scroll position 60 times/second
   - Creates smooth interpolation between scroll positions
   - Lenis calculates the difference between current and target scroll position
   - Each frame, it moves a fraction (lerp) closer to the target

**Visual Example:**
```
User scrolls 100px down:
Frame 1: Move 10px (10% of 100px)
Frame 2: Move 9px (10% of remaining 90px)
Frame 3: Move 8.1px (10% of remaining 81px)
... continues until reaching target
```

---

## 2. GSAP ScrollTrigger - The Core Animation Engine

### What is ScrollTrigger?
ScrollTrigger is a GSAP plugin that triggers animations based on scroll position. It watches elements and fires animations when they enter/exit the viewport.

### Key Concepts:

#### A. Trigger Points (Lines 335-343)
```javascript
ScrollTrigger.create({
    trigger: "#scrolly-section",
    start: "top center",     // When top of trigger hits center of viewport
    end: "bottom bottom",    // When bottom of trigger hits bottom of viewport
    onEnter: () => gsap.to(phoneWrapper, { opacity: 1, duration: 0.5 }),
    onLeave: () => gsap.to(phoneWrapper, { opacity: 0, duration: 0.5 }),
});
```

**How Trigger Points Work:**
```
Viewport:
┌─────────────────┐
│                 │ ← top
│     center      │ ← center (50%)
│                 │
└─────────────────┘ ← bottom

Trigger Element:
┌─────────────────┐ ← top of element
│   #scrolly-     │
│    section      │
└─────────────────┘ ← bottom of element

start: "top center" = When element's TOP crosses viewport's CENTER
end: "bottom bottom" = When element's BOTTOM crosses viewport's BOTTOM
```

#### B. Scrubbing (Lines 389-431)
```javascript
const finalTimeline = gsap.timeline({
    scrollTrigger: {
        trigger: "#final-trigger-spacer",
        start: "top center",
        end: "bottom center",
        scrub: 2,  // Links animation progress to scroll position
    }
});
```

**What `scrub` does:**
- `scrub: true` - Animation progress = scroll progress (instant)
- `scrub: 2` - Animation progress follows scroll with 2-second smooth delay
- Creates a "dragging" effect where you control animation speed by scrolling

**Example:**
```
Scroll Position:  0%  →  25%  →  50%  →  75%  →  100%
Animation:        ↓      ↓       ↓       ↓       ↓
Phone Rotation:   0°  →  45°  →  90°  → 135°  →  180°
```

---

## 3. The Scrollytelling Section - Step by Step

### Architecture (Lines 113-227)

The section has two main parts:
1. **Fixed Phone Mockup** (right side on desktop, background on mobile)
2. **Scrolling Text Cards** (left side, scrolls over phone)

### A. Phone Visibility Control (Lines 335-343)
```javascript
ScrollTrigger.create({
    trigger: "#scrolly-section",
    start: "top center",
    end: "bottom bottom",
    onEnter: () => gsap.to(phoneWrapper, { opacity: 1, duration: 0.5 }),
    onLeave: () => gsap.to(phoneWrapper, { opacity: 0, duration: 0.5 }),
});
```

**Flow:**
1. Phone starts invisible (`opacity: 0`)
2. When section enters viewport → fade in
3. When section leaves viewport → fade out
4. Callbacks: `onEnter`, `onEnterBack`, `onLeave`, `onLeaveBack`

### B. Step-by-Step Image Changes (Lines 346-383)

Each text card triggers a phone screen update:

```javascript
stepCards.forEach((step, index) => {
    ScrollTrigger.create({
        trigger: step,
        start: "top 70%",    // Trigger when card is 70% down viewport
        end: "bottom 30%",   // End when card is 30% down viewport
        onEnter: () => updatePhone(index),
        onEnterBack: () => updatePhone(index),
    });
});
```

**The `updatePhone()` Function:**
```javascript
function updatePhone(index) {
    // 1. Change screenshot
    phoneImages.forEach((img, i) => {
        if(i === index) {
            img.style.opacity = 1;           // Show this image
            img.style.transform = 'scale(1)';
        } else {
            img.style.opacity = 0;           // Hide others
            img.style.transform = 'scale(1.02)'; // Slight zoom for transition
        }
    });

    // 2. Change glow color
    glow.style.backgroundColor = colors[index] + '60'; // 60 = opacity

    // 3. Highlight active text card
    stepCards.forEach((card, i) => {
        if (i === index) {
            gsap.to(card, { opacity: 1, scale: 1 });
        } else {
            gsap.to(card, { opacity: 0.3, scale: 0.98 });
        }
    });
}
```

**Visual Flow:**
```
Scroll Down:
┌──────────────────┐
│ Step 1 Card      │ ← Active (opacity: 1, scale: 1)
│ [Phone shows     │
│  screenshot 1]   │
├──────────────────┤
│ Step 2 Card      │ ← Inactive (opacity: 0.3, scale: 0.98)
└──────────────────┘

User scrolls...

┌──────────────────┐
│ Step 1 Card      │ ← Inactive
├──────────────────┤
│ Step 2 Card      │ ← Active
│ [Phone shows     │
│  screenshot 2]   │
└──────────────────┘
```

### C. The Final Sequence - Phone Flip Animation (Lines 387-431)

This is the most complex part - a timeline that flips the phone, takes a photo, and shows a message.

```javascript
const finalTimeline = gsap.timeline({
    scrollTrigger: {
        trigger: "#final-trigger-spacer",  // 200vh tall spacer
        start: "top center",
        end: "bottom center",
        scrub: 2,  // Smooth 2-second lag
    }
});

finalTimeline
    // 1. Flip phone to show back (camera)
    .to("#phone-mockup", { 
        rotateY: 180,      // Flip 180 degrees
        scale: 0.8,        // Shrink slightly
        duration: 1.5,
        ease: "power2.inOut"
    }, 0)
    
    // 2. Hold on back for 0.5 seconds
    .to({}, { duration: 0.5 })
    
    // 3. Flash the camera LED
    .to("#camera-flash", { 
        backgroundColor: "#fff",
        boxShadow: "0 0 60px 20px #fff",
        duration: 0.25,
        yoyo: true,    // Go back to original
        repeat: 1      // Flash twice (original + repeat)
    })
    
    // 4. Flash the entire screen (synced with LED)
    .to("#screen-flash", {
        opacity: 0.9,
        duration: 0.25,
        yoyo: true,
        repeat: 1
    }, "<")  // "<" means start at same time as previous animation
    
    // 5. Hold on back for 1.2 seconds
    .to({}, { duration: 1.2 })
    
    // 6. Flip back to front
    .to("#phone-mockup", { 
        rotateY: 360,  // Complete the rotation (180 → 360)
        scale: 1,
        duration: 1.5,
        ease: "power2.inOut"
    })
    
    // 7. Show "Ready for Next Trip?" message
    .to("#ready-msg", {
        opacity: 1,
        scale: 1,
        duration: 0.8,
        ease: "back.out(1.7)"  // Bouncy effect
    }, "-=0.2");  // Start 0.2 seconds before previous animation ends
```

**Timeline Visualization:**
```
Scroll Progress:  0%        25%       50%       75%      100%
                  ↓          ↓         ↓         ↓         ↓
Animation:     [Flip to] [Hold] [Flash!] [Hold] [Flip] [Message]
                 back                           front    pops up
Phone Rotation:  0° → 180°  (hold)   (hold)   180° → 360°
```

**Key Timing Concepts:**
- `duration` in timeline = relative time units (not real seconds when scrubbing)
- Total timeline duration = sum of all durations
- Scroll distance maps to timeline progress
- `scrub: 2` adds smooth interpolation

---

## 4. Responsive Behavior with matchMedia

### Desktop-Specific Animation (Lines 305-322)
```javascript
mm.add("(min-width: 1024px)", () => {
    gsap.timeline({
        scrollTrigger: {
            trigger: finalTrigger,
            start: "top center",
            end: "bottom center",
            scrub: 2
        }
    }).to(phoneWrapper, {
        left: 0,           // Move from right side
        width: "100%",     // Expand to full width
        duration: 1.5,
    }, 0);
});
```

**What this does:**
- On desktop: Phone starts on right side, moves to center during final sequence
- On mobile: Phone stays centered (this animation doesn't run)

### Global Animations (Lines 325-433)
```javascript
mm.add("(min-width: 1px)", () => {
    // All the phone visibility, image switching, and flip animations
});
```
- Runs on all screen sizes (hence `1px` minimum)
- Contains the core scrollytelling logic

---

## 5. CSS 3D Transforms

### The Phone Flip Effect (Lines 128-165)

The phone has two faces (front and back) using CSS 3D transforms:

```html
<div id="phone-mockup" style="transform-style: preserve-3d;">
    <!-- FRONT FACE -->
    <div style="transform: rotateY(0deg); backface-visibility: hidden;">
        <!-- Screenshots go here -->
    </div>
    
    <!-- BACK FACE -->
    <div style="transform: rotateY(180deg); backface-visibility: hidden;">
        <!-- Camera design goes here -->
    </div>
</div>
```

**How it works:**
1. `transform-style: preserve-3d` - Enables 3D space for children
2. `backface-visibility: hidden` - Hides face when rotated away
3. Front starts at `rotateY(0deg)` - visible
4. Back starts at `rotateY(180deg)` - hidden (facing away)
5. When parent rotates to 180°:
   - Front rotates to 180° (now facing away) → hidden
   - Back rotates to 360° (0°) (now facing forward) → visible

**Visual:**
```
Initial State (rotateY: 0°):
Front: 0° → visible ✓
Back: 180° → hidden ✗

After Rotation (rotateY: 180°):
Front: 180° → hidden ✗
Back: 360° (0°) → visible ✓
```

---

## 6. Performance Optimizations

### 1. CSS Transitions for Simple Animations
```css
.screen-img {
    transition: opacity 500ms ease-in-out;
}
```
- Browser-optimized (GPU accelerated)
- Used for simple opacity changes

### 2. Transform-Only Animations
```javascript
gsap.to(card, { opacity: 1, scale: 1 });  // scale = transform: scale()
```
- `transform` and `opacity` are GPU-accelerated
- Avoid animating `width`, `height`, `top`, `left` (causes reflow)

### 3. RequestAnimationFrame Loop
```javascript
function raf(time) { 
    lenis.raf(time); 
    requestAnimationFrame(raf); 
}
```
- Syncs with browser's 60fps refresh rate
- Pauses when tab is inactive (saves CPU)

### 4. Scrub Smoothing
```javascript
scrub: 2  // 2-second smooth delay
```
- Prevents janky animations from scroll input
- Interpolates between scroll positions

---

## 7. Key Takeaways

### How Smooth Scrolling Works:
1. **Lenis intercepts scroll events**
2. **Calculates target scroll position**
3. **Interpolates current position toward target** (lerp)
4. **Updates 60 times/second** (requestAnimationFrame)
5. **Result: Smooth, eased scrolling instead of instant jumps**

### How GSAP ScrollTrigger Works:
1. **Watches scroll position** (synced with Lenis)
2. **Compares to trigger points** (start/end)
3. **Fires callbacks** (onEnter, onLeave, etc.)
4. **With scrub: Maps scroll progress to animation progress**

### The Scrollytelling Pattern:
1. **Fixed element** (phone) with `position: fixed`
2. **Scrolling content** (text cards) with `position: relative`
3. **ScrollTrigger watches cards** and updates phone
4. **Creates illusion** that scrolling controls the phone

### Why It Feels Smooth:
- **Lenis lerp** creates momentum-based scrolling
- **GSAP scrub** smooths animation playback
- **GPU-accelerated transforms** (scale, rotate, opacity)
- **60fps requestAnimationFrame** loop
- **Careful easing functions** (power2.inOut, back.out)

---

## 8. Common GSAP Patterns Used

### Pattern 1: Fade In/Out
```javascript
gsap.to(element, { opacity: 1, duration: 0.5 });
```

### Pattern 2: Scale & Rotate
```javascript
gsap.to(element, { scale: 0.8, rotateY: 180, duration: 1.5 });
```

### Pattern 3: Yoyo (Bounce Back)
```javascript
gsap.to(element, { opacity: 1, yoyo: true, repeat: 1 });
// Goes: 0 → 1 → 0 (original → target → original)
```

### Pattern 4: Timeline Sequencing
```javascript
timeline
    .to(el1, { x: 100 }, 0)      // Start at 0 seconds
    .to(el2, { y: 50 }, 0.5)     // Start at 0.5 seconds
    .to(el3, { scale: 2 }, "<")  // Start with previous
    .to(el4, { opacity: 0 }, "-=0.2"); // Start 0.2s before previous ends
```

### Pattern 5: Scrubbed Timeline
```javascript
gsap.timeline({
    scrollTrigger: {
        trigger: element,
        scrub: 2,  // Scroll controls playback
    }
})
```

---

## Debugging Tips

### 1. Visualize ScrollTrigger
```javascript
ScrollTrigger.create({
    markers: true,  // Shows start/end lines
    trigger: element,
    start: "top center",
    end: "bottom center",
});
```

### 2. Log Animation Progress
```javascript
gsap.to(element, {
    x: 100,
    onUpdate: function() {
        console.log(this.progress());  // 0 to 1
    }
});
```

### 3. Check Lenis Scroll Position
```javascript
lenis.on('scroll', ({ scroll, velocity }) => {
    console.log('Scroll:', scroll, 'Velocity:', velocity);
});
```

---

## Summary

This landing page combines:
- **Lenis** for smooth, interpolated scrolling (the "heavy" feel)
- **GSAP** for powerful, performant animations
- **ScrollTrigger** to sync animations with scroll position
- **CSS 3D transforms** for the phone flip effect
- **Responsive design** with matchMedia for desktop/mobile variants

The result is a cinematic, interactive experience where scrolling feels like controlling a video timeline.
