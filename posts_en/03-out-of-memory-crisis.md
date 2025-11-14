# 🔥 Stage 3: Out of Memory Crisis

**Date:** November 2025  
**Topic:** Investigation and fixing critical bug

[🇷🇺 Русский](../posts/03-out-of-memory-crisis.md) | **🇬🇧 English**

---

## 💥 The Problem

Advanced Manager worked great for several days. But suddenly:

```
[CRASH]
Fatal error: Ran out of memory allocating 1.0 MiB
Last error msg: Pagefile is too small to complete operation.

Unhandled Exception: 0x0000c000
Crash in runnable thread IOThreadPool #1
```

**Server crashed!** 😱

---

## 🔍 Investigation

### First thought: "Not enough RAM?"

Check:
- RAM: 16 GB
- PageFile: 16 GB
- Players online: 3

**Should be enough...**

### Second thought: "Memory leak in game?"

Checking logs:
- Server ran for 6 hours
- Several restarts
- Crash 2-3 hours after each restart

**Strange...** 🤔

### The Discovery!

Player writes:
> "I checked Task Manager and found **multiple copies** of running server!"

**Bingo!** 🎯

---

## 🐛 The Cause

Turns out:

```
1. Server starts → 2 processes (wrapper + actual)
2. Restart → old processes not fully killed
3. New server starts → +2 processes
4. Total: 4 processes instead of 2!
5. Another restart → 6 processes
6. Another restart → 8 processes
...
10. RAM and PageFile run out
11. Crash!
```

**Problem was in the manager, not the game!** 😅

---

## 🔧 The Fix

Needed to fix 5 bugs in the manager:

### 1. Incomplete process killing
**Was:** Only main process killed  
**Now:** ALL child processes killed recursively

### 2. Insufficient restart delay
**Was:** 3 seconds between stop and start  
**Now:** 10 seconds + verification all killed

### 3. No check before start
**Was:** Just started new process  
**Now:** First check - any already running?

### 4. No control on manager exit
**Was:** Closed manager → server left running, processes accumulate  
**Now:** On exit asks what to do with processes

### 5. No visual control
**Was:** Can't see how many processes actually running  
**Now:** Process counter in interface

---

## ✅ Result

After fixes:

```
✅ Always only 2 processes
✅ After restart - new PIDs
✅ Old processes completely killed
✅ No more Out of Memory
✅ Server runs stable for days
```

---

## 📊 Testing

Stress test conducted:
- 20+ consecutive restarts
- Task Manager check after each
- Memory monitoring

**Result:** Always 2 processes, no memory leak! ✅

---

## 💡 Lessons Learned

### What we learned:

1. **Problem not always where it seems**
   - Thought: game leaks memory
   - Reality: manager wasn't killing processes

2. **Importance of testing**
   - Basic tests passed
   - But long-term use revealed bug

3. **Feedback is critical**
   - Player reported multiple processes
   - This gave direction for search

4. **Automation must be done right**
   - Automatic restarts - good
   - But if poorly implemented - worse than manual

---

## 🎯 Conclusion

**Advanced Manager v2.1** (with fixes):
- All bugs fixed
- Works stable
- Ready for production use

---

## 🚀 But The Story Didn't End There...

Players started asking:
- "Can we see player statistics?"
- "How to know who killed most zombies?"
- "Can backups be automatic?"

**Idea:** Create Ultimate Manager - all in one!

**Continuation:** [Stage 4: HTTP API and New Horizons →](./04-http-api-discovery.md)

---

**Stage 3 completed:** Critical bug defeated! 🔥→✅

