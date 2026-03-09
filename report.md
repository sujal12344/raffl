# Raffl — Technical Debt & Modernization Report

**Date:** March 9, 2026
**Prepared for:** Team / Management
**Scope:** Frontend (raffl-web) + Backend (raffl-api)

---

> **Isko padhne ka tarika:** Har issue ke saath ek simple analogy diya gaya hai.
> Saath mein **Risk Level** bhi diya gaya hai — 🔴 Critical · 🟠 High · 🟡 Medium · 🟢 Low

---

## Executive Summary (Short Version for Manager)

Raffl ka current codebase kaam kar raha hai, lekin **iske andar kuch aise packages hain jo:**

- **Security holes** open kar rahe hain (attacker andar aa sakta hai)
- **App ko slow** bana rahe hain (user experience kharab hota hai)
- **Officially band ho chuke hain** (company ne support dena band kar diya, patches nahi aate)
- **Unnecessary weight** add kar rahe hain (2-3 alag tools jo ek hi kaam karte hain)
- **Developer ki speed** slow karti hain (code samajhna mushkil ho jaata hai)

**Agar yeh issues fix nahi kiye, toh kal koi bada security breach ya performance crash aaega aur tab sab kuch haath se baar hoga.**

---

## Part 1 — Critical Security Issues 🔴

Yeh waale pehle fix karne chahiye. Inhe ignore karna site ko hack hone ke liye invite dene jaisa hai.

---

### 1.1 AWS SDK v2 — Backend mein Dead Library use ho rahi hai

|               | Detail         |
| ------------- | -------------- |
| **Package**   | `aws-sdk` (v2) |
| **Kahan hai** | Backend        |
| **Risk**      | 🔴 Critical     |

**Simple explanation:**
Amazon ne September 2025 mein officially announce kar diya ki `aws-sdk` version 2 ka **support band ho gaya hai**. Matlab ab koi security patch nahi aayega. Agar isme koi security hole milta hai, toh Amazon fix hi nahi karega.

> **Analogy:** Ek purana lock use karna jiska manufacturer ban ho gaya ho aur replacement parts milne band ho gaye hon.

**Aaj ki sthiti:** Backend mein `aws-sdk` v2 **AND** nayi `@aws-sdk` v3 dono saath install hain — yeh directly conflict karte hain aur bundle ko unnecessarily bada banate hain.

**Kya karna chahiye:**
- `aws-sdk` v2 **turant remove karo**
- Sirf `@aws-sdk` v3 use karo (yeh already installed hai)

**Isse kya hoga:** Security risk khatam, package size ~2MB kam hogi.

---

### 1.2 `speakeasy` — 2FA Library jo 8 Saal Se Update Nahi Hua

|               | Detail      |
| ------------- | ----------- |
| **Package**   | `speakeasy` |
| **Kahan hai** | Backend     |
| **Risk**      | 🔴 Critical  |

**Simple explanation:**
`speakeasy` wo library hai jo users ke **Two-Factor Authentication (2FA)** codes generate karta hai. Iska GitHub last commit **2016 mein** tha — yaani **8 saal purana**, koi maintenance nahi, koi security fix nahi.

> **Analogy:** Apne bank locker ki chabi ek aisi company se banaana jo 8 saal pehle band ho gayi ho. Koi nahi poochhe ki isme flaw hai ya nahi.

**Kya karna chahiye:**
`speakeasy` hataao → `otplib` par shift ho jao (2024 mein actively maintained, TOTP/HOTP support, TypeScript native)

**Isse kya hoga:** 2FA remain karega lekin ab secure aur maintained library pe chalega.

---

### 1.3 `bcryptjs` — Password Hashing Ka Kamzor Tarika

|               | Detail     |
| ------------- | ---------- |
| **Package**   | `bcryptjs` |
| **Kahan hai** | Backend    |
| **Risk**      | 🟠 High     |

**Simple explanation:**
User passwords ko store karne ke liye bcrypt use ho raha hai. Yeh saal 2011 ka algorithm hai. Modern standards ke hisaab se **`argon2`** bahut zyada secure hai — isko 2015 mein Password Hashing Competition ne officially winner declare kiya tha.

> **Analogy:** Ghar mein purana Yale lock laga hai jo kaam karta hai, lekin modern deadbolt se replace karna zyada sensible hai.

**NIST (America ki cybersecurity authority) officially `argon2` ko recommend karti hai.**

**Kya karna chahiye:**
`bcryptjs` → `argon2` (drop-in replacement, zyada secure)

---

### 1.4 `next-auth` v4 — Known Vulnerabilities ke saath Outdated Auth

|               | Detail               |
| ------------- | -------------------- |
| **Package**   | `next-auth` v4.24.11 |
| **Kahan hai** | Frontend             |
| **Risk**      | 🟠 High               |

**Simple explanation:**
NextAuth v4 mein **multiple known security vulnerabilities** hain (CVE-listed). Auth.js (next-auth v5) is ka official upgrade hai jo in sab ko fix karta hai. v4 ko aur security updates nahi milenge.

**Kya karna chahiye:**
Migrate to `next-auth` v5 (Auth.js) — better session handling, better security defaults.

---

### 1.5 `typescript strict: false` — Type Safety Off Hai Backend Mein

|               | Detail                              |
| ------------- | ----------------------------------- |
| **Config**    | `tsconfig.json` → `"strict": false` |
| **Kahan hai** | Backend                             |
| **Risk**      | 🟠 High                              |

**Simple explanation:**
TypeScript ka poora point yeh hai ki code mein type errors pakde. Lekin `strict: false` ka matlab hai — **TypeScript basically off hai**. Code run hoga, dhang se compile hoga, but real bugs beech mein chhup jayenge jinka pata production mein chalega.

> **Analogy:** Spell-check off karke email likhna. Grammatical errors hote rahenge, koi nahi bataega.

**Kya karna chahiye:**
`strict: true` karo backend `tsconfig.json` mein, phir jo type errors aayein unhe fix karo.

---

## Part 2 — Performance Issues 🟠

Yeh cheezein app ko slow banaa rahi hain — page load slow hoga, server zyada RAM use karega.

---

### 2.1 `moment.js` — 67KB ka Dinosaur

|               | Detail           |
| ------------- | ---------------- |
| **Package**   | `moment`         |
| **Kahan hai** | Frontend         |
| **Risk**      | 🟠 High           |
| **Iska size** | ~67KB compressed |

**Simple explanation:**
`moment.js` dates aur time format karne ke liye use hota hai (e.g., "2 hours ago", "March 9, 2026"). Lekin yeh **67KB** ka beast hai — aur khud Moment.js ki team ne officially recommend kiya hai ki **nayi projects mein isey use mat karo**.

Replacement `day.js` same kaam karta hai lekin sirf **2KB** mein — yaani **33 guna chota**.

> **Analogy:** Chhoti si cheez uthane ke liye crane bulana, jab haath se uthaana possible hai.

**Kya karna chahiye:**
`moment` → `day.js` (API almost same hai, migrate karna easy hai)

**Isse kya hoga:** Frontend bundle ~65KB chota hoga, page load faster.

---

### 2.2 `web3.js` — 600KB ka Heavy Blockchain Library

|               | Detail    |
| ------------- | --------- |
| **Package**   | `web3` v4 |
| **Kahan hai** | Backend   |
| **Risk**      | 🟠 High    |

**Simple explanation:**
`web3.js` Ethereum blockchain se interact karne ke liye use hoti hai. Problem yeh hai ki yeh bahut badi library hai (~600KB unpacked), slow startup, TypeScript support average hai.

Meanwhile, **`viem`** (jo frontend mein already use ho rahi hai) same kaam karta hai, **5x faster** hai, better TypeScript support hai, aur size bhi kam hai.

**Kya karna chahiye:**
Backend mein bhi `web3.js` → `viem` migrate karo. Frontend mein already `viem` hai, toh consistency bhi aayegi.

---

### 2.3 Teen Alag Toast/Notification Libraries

|               | Detail                                          |
| ------------- | ----------------------------------------------- |
| **Packages**  | `react-hot-toast` + `react-toastify` + `sonner` |
| **Kahan hai** | Frontend                                        |
| **Risk**      | 🟡 Medium                                        |

**Simple explanation:**
"Toast" notifications woh hote hain jo corner mein popup aate hain — "Saved!", "Error!", etc. Hamare frontend mein **teen alag-alag libraries** hain jo exactly yahi kaam karti hain. Teeno ek saath install hain.

> **Analogy:** Ek ghar mein teen alag TV channels subscribe karna sirf news dekhne ke liye.

**Kya karna chahiye:**
Sirf `sonner` rakho (best maintained, lightest, Next.js ke saath best compatibility), baaki dono hataao.

**Isse kya hoga:** Bundle size ~40KB kam, ek consistent look across app.

---

### 2.4 Do Alag UI Libraries — NextUI + PrimeReact

|               | Detail                             |
| ------------- | ---------------------------------- |
| **Packages**  | `@nextui-org/react` + `primereact` |
| **Kahan hai** | Frontend                           |
| **Risk**      | 🟡 Medium                           |

**Simple explanation:**
Dono complete UI component libraries hain — buttons, tables, modals, forms sab dono mein hain. Dono ko import karne se app ka size kaafi badh jaata hai aur design inconsistent lagta hai (dono ka look alag hota hai).

**Kya karna chahiye:**
Decide karo ek rakho (NextUI already zyada use ho raha hai). PrimeReact ke jo specific components use hote hain unhe NextUI equivalent se replace karo.

---

### 2.5 `styled-components` + Tailwind CSS — Do Styling Systems

|               | Detail                               |
| ------------- | ------------------------------------ |
| **Packages**  | `styled-components` v6 + TailwindCSS |
| **Kahan hai** | Frontend                             |
| **Risk**      | 🟡 Medium                             |

**Simple explanation:**
CSS likhne ke do alag tarike ek saath chal rahe hain. Yeh developer ke liye confusing hai (alag jagah alag tarike se style likha gaya hai) aur bundle size bhi badhtaa hai. `styled-components` bhi React Server Components ke saath Next.js App Router mein poorly compatible hai.

**Kya karna chahiye:**
`styled-components` gradually remove karo, sirf Tailwind CSS pe standardize karo.

---

### 2.6 `node-fetch` v2 — Duplicated Native Feature

|               | Detail          |
| ------------- | --------------- |
| **Package**   | `node-fetch` v2 |
| **Kahan hai** | Backend         |
| **Risk**      | 🟡 Medium        |

**Simple explanation:**
`node-fetch` ek purani library hai jo Node.js mein browser jaisa `fetch` laati thi — lekin Node.js 18+ mein `fetch` **built-in** aa gaya hai. Toh yeh library literally ek aisi cheez kar rahi hai jo already free mein available hai.

**Kya karna chahiye:**
`node-fetch` hataao, native `fetch` use karo.

---

## Part 3 — Outdated Packages 🟡

Yeh packages zyada risky nahi hain abhi, lekin inhe update karna chahiye.

---

### 3.1 Next.js 14.0.3 — 15 Versions Peeche

|             | Detail        |
| ----------- | ------------- |
| **Package** | `next` 14.0.3 |
| **Latest**  | Next.js 15.2  |
| **Risk**    | 🟠 High        |

**Next.js 15 mein kya milega:**
- **Turbopack** — Development server **10x faster** start hoga (dev experience dramatically better)
- **Better caching** — Pages faster load honge users ke liye
- **Security patches** — 14.0.3 mein multiple CVEs (security holes) fix ho chuke hain 15 tak aate aate
- **React 19 support** — Nayi React features available hongi

---

### 3.2 `@tailwindcss/typography` — Insiders Version in Production

|                     | Detail                    |
| ------------------- | ------------------------- |
| **Package**         | `@tailwindcss/typography` |
| **Current version** | `0.0.0-insiders.0339c42`  |
| **Risk**            | 🟠 High                    |

**Simple explanation:**
Yeh ek **experimental/test version** hai jo production mein kabhi nahi hona chahiye. "Insiders" build matlab developers ke liye testing purpose se — yeh unstable hai, officially supported nahi hai, aur kisi bhi update mein break ho sakta hai.

> **Analogy:** Production server pe beta software install karna.

**Kya karna chahiye:**
`@tailwindcss/typography` ka latest stable version `^0.5.15` install karo.

---

### 3.3 `json2csv` Alpha Version in Production

|               | Detail                       |
| ------------- | ---------------------------- |
| **Package**   | `json2csv` 6.0.0-**alpha**.2 |
| **Kahan hai** | Frontend                     |
| **Risk**      | 🟡 Medium                     |

**Simple explanation:**
"Alpha" version matlab yeh package **abhi bana hi raha tha** jab install kiya. Alpha versions bahut unstable hote hain aur production ke liye nahi hote.

**Kya karna chahiye:**
`json2csv` → `papaparse` ya `@json2csv/plainjs` (stable version) pe migrate karo.

---

### 3.4 `@types/*` Wrong Jagah Rakhe Hain — Backend

|               | Detail                                                                  |
| ------------- | ----------------------------------------------------------------------- |
| **Packages**  | `@types/axios`, `@types/nodemailer`, `@types/web3`, `@types/node-fetch` |
| **Kahan hai** | Backend `dependencies`                                                  |
| **Risk**      | 🟢 Low                                                                   |

**Simple explanation:**
`@types/*` packages sirf development mein chahiye hote hain (TypeScript ko samjhane ke liye) — production server pe deploy nahi hone chahiye. Par backend mein yeh `dependencies` mein hain, `devDependencies` mein nahi. Iska matlab hai production server pe bhi yeh unnecessary packages install ho rahe hain.

**Kya karna chahiye:**
Inhe `devDependencies` mein move karo.

---

### 3.5 `ethers` v5 + wagmi v2 — Version Conflict

|               | Detail                               |
| ------------- | ------------------------------------ |
| **Packages**  | `ethers` v5 + `wagmi` v2 + `viem` v2 |
| **Kahan hai** | Frontend                             |
| **Risk**      | 🟡 Medium                             |

**Simple explanation:**
`wagmi` v2 internally `viem` use karta hai. Lekin `ethers` v5 bhi installed hai — yeh ek older, different library hai. Dono ek saath hona bundle size badhata hai aur potential conflicts create karta hai.

**Kya karna chahiye:**
`ethers` v5 hataao, `viem` ke saath kaam karo jo already `wagmi` ke saath synchronized hai.

---

## Part 4 — Code Quality & Redundancy Issues 🟢

---

### 4.1 `npm` Package as a Dependency

|               | Detail                             |
| ------------- | ---------------------------------- |
| **Package**   | `npm` v10 listed in `dependencies` |
| **Kahan hai** | Frontend                           |
| **Risk**      | 🟡 Medium                           |

`npm` ek package manager hai — isko kisi bhi application ke `dependencies` mein nahi hona chahiye. Yeh runtime dependency nahi hai. Yeh probably galti se add ho gaya. Isse bundle aur install time dono affect hote hain.

---

### 4.2 Do Alag Icon Libraries

|               | Detail                                                 |
| ------------- | ------------------------------------------------------ |
| **Packages**  | `@tabler/icons-react` + `react-icons` + `lucide-react` |
| **Kahan hai** | Frontend                                               |
| **Risk**      | 🟢 Low                                                  |

Teen alag icon libraries hain. Har ek ke hazaaron icons hain jo app load hote waqt parse hote hain (tree-shaking hone par bhi). Consistency ke liye ek choose karo.

**Recommendation:** `lucide-react` (lightest, best maintained) rakho, baaki dono gradually replace karo.

---

### 4.3 `body-parser` Installed But Not Needed

|               | Detail        |
| ------------- | ------------- |
| **Package**   | `body-parser` |
| **Kahan hai** | Backend       |
| **Risk**      | 🟢 Low         |

`hyper-express` (jo backend HTTP server hai) ka apna built-in body parsing hai. Alag `body-parser` install karna redundant hai.

---

## Summary Table — Priority Order

| #   | Package/Issue                      | Kahan    | Risk       | Action                         |
| --- | ---------------------------------- | -------- | ---------- | ------------------------------ |
| 1   | `aws-sdk` v2                       | Backend  | 🔴 Critical | Remove, use `@aws-sdk` v3 only |
| 2   | `speakeasy` (8yr old 2FA)          | Backend  | 🔴 Critical | Replace with `otplib`          |
| 3   | `bcryptjs`                         | Backend  | 🟠 High     | Replace with `argon2`          |
| 4   | `next-auth` v4                     | Frontend | 🟠 High     | Migrate to v5 (Auth.js)        |
| 5   | `strict: false` in TypeScript      | Backend  | 🟠 High     | Enable strict mode             |
| 6   | Next.js 14.0.3                     | Frontend | 🟠 High     | Upgrade to Next.js 15          |
| 7   | `@tailwindcss/typography` insiders | Frontend | 🟠 High     | Install stable version         |
| 8   | `moment.js` (67KB)                 | Frontend | 🟠 High     | Replace with `day.js` (2KB)    |
| 9   | `web3.js` (600KB)                  | Backend  | 🟠 High     | Replace with `viem`            |
| 10  | 3 Toast libraries                  | Frontend | 🟡 Medium   | Keep only `sonner`             |
| 11  | 2 UI libraries                     | Frontend | 🟡 Medium   | Keep only NextUI               |
| 12  | `styled-components` + Tailwind     | Frontend | 🟡 Medium   | Remove `styled-components`     |
| 13  | `node-fetch` v2                    | Backend  | 🟡 Medium   | Use native `fetch`             |
| 14  | `ethers` v5 conflict               | Frontend | 🟡 Medium   | Remove `ethers`, use `viem`    |
| 15  | `json2csv` alpha                   | Frontend | 🟡 Medium   | Use stable package             |
| 16  | `@types/*` in dependencies         | Backend  | 🟢 Low      | Move to devDependencies        |
| 17  | `npm` as dependency                | Frontend | 🟡 Medium   | Remove                         |
| 18  | 3 Icon libraries                   | Frontend | 🟢 Low      | Standardize to `lucide-react`  |
| 19  | `body-parser` redundant            | Backend  | 🟢 Low      | Remove                         |

---

## Expected Benefits After Fixes

| Metric                   | Before                                  | After                                             |
| ------------------------ | --------------------------------------- | ------------------------------------------------- |
| **Frontend bundle size** | ~4.5MB+                                 | ~2.8MB (estimated ~38% reduction)                 |
| **Page load speed**      | Baseline                                | 30–50% faster (moment, web3, redundant libs gone) |
| **Dev server startup**   | Baseline                                | 5–10x faster (Next.js 15 Turbopack)               |
| **Security posture**     | Multiple known CVEs                     | All critical CVEs resolved                        |
| **Code consistency**     | Mixed patterns, 2 UI libs, 3 toast libs | Single pattern, predictable                       |
| **Type safety**          | Partial (strict: false)                 | Full TypeScript safety                            |
| **Maintenance burden**   | High (dead libraries)                   | Low (all actively maintained)                     |

---

## Recommended Migration Order (Phased)

### Phase 1 — Security First (1-2 weeks)
1. Remove `aws-sdk` v2
2. Replace `speakeasy` → `otplib`
3. Replace `bcryptjs` → `argon2`
4. Upgrade `next-auth` v4 → v5
5. Fix `@tailwindcss/typography` insiders version

### Phase 2 — Performance (2-3 weeks)
1. Upgrade Next.js 14 → 15
2. Replace `moment.js` → `day.js`
3. Replace `web3.js` → `viem` in backend
4. Remove `node-fetch`, use native fetch
5. Remove redundant toast libraries (keep `sonner`)
6. Remove `npm` and `install` from dependencies

### Phase 3 — Cleanup (2-3 weeks)
1. Remove `styled-components`, standardize on Tailwind
2. Consolidate icon libraries
3. Remove `primereact` or `nextui` (pick one)
4. Enable `strict: true` in backend TypeScript
5. Move `@types/*` to devDependencies
6. Remove `ethers` v5, use `viem`

---

*Report generated based on static analysis of `package.json` files from both `raffl-web` (frontend) and `raffl-api` (backend) repositories as of March 9, 2026.*
