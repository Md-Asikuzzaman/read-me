# Git — Change Any Commit Message

এই guide দিয়ে Git history-এর যেকোনো commit-এর **commit message** change করা যাবে।

## Step 1 — Project folder-এ যাও

```bash
cd your-project-folder
```

সঠিক branch-এ আছো কিনা check করো:

```bash
git branch
```

---

## Step 2 — Interactive Rebase শুরু করো

```bash
git rebase -i --root
```

এতে Git history editor-এ খুলবে।

Example:

```text
pick cbea3562 Initial setup
pick abc1234 Add navbar
pick def5678 Add filtering
pick ghi9012 Fix mobile UI
```

---

## Step 3 — `pick` → `reword` করো

যেই commit-এর message change করতে চাও, সেটা খুঁজে বের করো।

Example:

```text
pick cbea3562 Initial setup
```

এটাকে change করো:

```text
reword cbea3562 Initial setup
```

**অন্য commit-এর line change করবে না।**

> ⚠️ `pick` কোনো terminal command না। এটা শুধু rebase editor-এর ভিতরের instruction।

---

## Step 4 — নতুন Commit Message দাও

Save করে editor বন্ধ করার পর Git নতুন commit message চাইবে।

Example:

```text
Initial commit
```

Save করে close করো।

তারপর history check করো:

```bash
git log --oneline --reverse
```

তোমার updated commit message দেখতে পাবে।

> ⚠️ Commit message change করলে ওই commit-এর **hash পরিবর্তন হবে**। এটা normal।

---

## Step 5 — GitHub-এ Push করো

যেহেতু history rewrite হয়েছে, normal push কাজ নাও করতে পারে।

Use:

```bash
git push --force-with-lease origin main
```

`--force-with-lease` সাধারণ `--force`-এর চেয়ে safer।

---

# Example — First Commit Change

ধরো তোমার first commit:

```text
cbea3562 Initial setup
```

### 1. Run:

```bash
git rebase -i --root
```

### 2. Change:

```text
pick cbea3562 Initial setup
```

to:

```text
reword cbea3562 Initial setup
```

### 3. New message:

```text
Initial commit
```

### 4. Check:

```bash
git log --oneline --reverse
```

### 5. Push:

```bash
git push --force-with-lease origin main
```

---

# Quick Cheat Sheet

```bash
# Step 1
cd your-project-folder

# Step 2
git rebase -i --root

# Step 3
# In the editor:
# pick COMMIT_ID Old message
# ↓
# reword COMMIT_ID Old message

# Step 4
# Enter the new commit message

# Step 5
git push --force-with-lease origin main
```

## Important

* `reword` → commit message change করে
* `edit` → commit-এর content modify করার জন্য
* পুরোনো commit change করলে Git history rewrite হয়
* Changed commit-এর hash নতুন হবে
* পরের commitগুলোর hash-ও পরিবর্তিত হতে পারে
* `pick` terminal-এ লিখবে না
* Shared branch হলে history rewrite করার আগে অন্যদের জানানো উচিত
* `--force-with-lease` ব্যবহার করা `--force` থেকে safer
