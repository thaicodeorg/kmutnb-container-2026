# 📋 Summary: `docker diff` → `docker commit` → `docker history`

## 🔍 What Happened

| Command | Output | Meaning |
|---|---|---|
| `docker diff overlay-test` | `C /root`, `A /root/.bash_history`, `A /test`, `A /test/file.txt` | Shows **all changes** the container made in its writable layer (`upperdir`) |
| `docker commit overlay-test overlay-test-snapshot` | `sha256:9c208eed6f06...` | **Freezes** the container's writable layer into a **new read-only image layer** |
| `docker history overlay-test-snapshot` | Top row: `9c208eed6f06` = **8.19 kB**; below: `ad14f7d919c9` = **329 MB** (original CentOS base) | Confirms the new image = **tiny new layer + shared base image layers** |

## 📊 Diff Flags Explained

| Flag | Meaning | Example |
|---|---|---|
| `A` | **Added** | `/test`, `/test/file.txt` |
| `C` | **Changed** | `/root` (dir metadata) |
| `D` | **Deleted** | (none here) |

## 🎨 Before vs. After Commit

```
BEFORE commit:                     AFTER commit:
┌────────────────────┐             ┌────────────────────┐
│ Container (RW)     │             │ NEW IMAGE          │
│  /test/file.txt    │  ──commit──▶│  layer: 8.19 kB   │ ← frozen upperdir
├────────────────────┤             ├────────────────────┤
│ CentOS base (RO)   │             │ CentOS base: 329MB │ ← SHARED, not copied
│  329 MB            │             └────────────────────┘
└────────────────────┘
```

## 🔑 Key Takeaways

- **`docker diff`** = shows what changed in the container's writable layer.
- **`docker commit`** = turns the writable layer into a new read-only image layer.
- **Base layers are shared** — the 329 MB CentOS base is **not duplicated**.
- **Only the delta (8.19 kB)** is stored as the new layer.
- **Every commit = one new layer** stacked on top of the previous ones.

---

## 🧠 3 Questions to Help You Understand

### ❓ Question 1: The "Shared Base" Question
**After committing, you have two images based on CentOS Stream 10. Does the 329 MB base layer get duplicated on disk?**

> **Hint:** No. Both images **share the same read-only lower layers**. Only the new 8.19 kB delta is added. This is the efficiency of overlayfs + content-addressable storage — layers are deduplicated by hash.

---

### ❓ Question 2: The "Where Did the 329 MB Go?" Question
**Your `docker diff` showed 4 changes, but the new commit layer is only 8.19 kB. Where did the 329 MB CentOS base go?**

> **Hint:** It wasn't captured — it was **inherited**. `docker commit` only stores the **upperdir delta** (your file + bash history + dir metadata). The 329 MB lives in the **lowerdirs** and is **referenced, not copied**.

---

### ❓ Question 3: The "Commit Again" Question
**Your container `overlay-test` still exists. If you write a new file inside it and commit again, what happens to the first snapshot `overlay-test-snapshot`?**

> **Hint:** **Nothing** — it's immutable. Each commit creates a **new independent image** stacked on top of the previous one. Layers are never modified after creation, only added to. This is how image layer chains work.

---

## 🔑 One-Line Memory Hook

> **"`diff` shows what changed. `commit` freezes the change into a new layer. Base layers are shared, never copied. Every commit adds a layer; nothing is modified in place."**