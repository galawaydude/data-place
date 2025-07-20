<%*
const contestNumber = await tp.system.prompt("Contest number (e.g. 404)")
const contestDate = tp.date.now("YYYY-MM-DD")
const totalProblems = 4  // change if needed
%>

# 📘 LeetCode Weekly Contest <%= contestNumber %>

> _Date: <%= contestDate %>_

---

<% for (let i = 1; i <= totalProblems; i++) { 
  const title = await tp.system.prompt(`Problem ${i} Title`)
  const difficulty = await tp.system.prompt(`Problem ${i} Difficulty (Easy/Medium/Hard)`)
  const lang = await tp.system.prompt(`Language for Problem ${i} Code`)
%>

## 📝 Problem <%= i %>: <%= title %>

**Difficulty**: <%= difficulty %>  
**Link**: [<%= title %>](https://leetcode.com/problems/<%= title.toLowerCase().replaceAll(" ", "-").replaceAll(/[^a-z0-9\-]/g, "") %>)

---

### 🔍 Logic

```text
Describe your approach here:
- What's the brute force idea?
- What's the optimized approach?
- Time and space complexity?
```

---

### 💡 Notes

```text
Any insights, mistakes, alternate approaches, or edge cases.
Write down anything you learned or want to remember.
```

---

### 💻 Code (<%= lang %>)

```<%= lang %>
// your solution here
```

---

<% } %>

## 🏁 Final Notes

```text
- Rank: 
- Score: 
- What went well?
- What to improve for next time?
```

---

## 🏷️ Tags

`#leetcode` `#contest` `#contest-<%= contestNumber %>` `#competitive-programming`
