# 🚀 Holiday Events - Jenkins CI/CD & Ansible Infrastructure

מערכת אוטומציה שלמה (CI/CD Pipeline) ותשתית כקוד (IaC) עבור אפליקציית Web מבוססת Node.js.  
הפרויקט מיישם הפרדת סביבות מלאה, בדיקות איכות אוטומטיות, ניהול קונטיינרים, ומנגנוני התאוששות מאסון בזמן אמת.

---

## 📐 ארכיטקטורת המערכת וזרימת נתונים (System Flow)

```text
  [עמדת מפתח - Git]
          │
    git push (main)
          ▼
   [GitHub Repository]
          │
    SCM Polling (אוטומטי)
          ▼
  [Jenkins CI/CD Engine]
   ├── 1. Checkout (משיכת קוד מקור)
   ├── 2. Install & Test (בדיקות תלויות ואיכות)
   ├── 3. Build & Tag (תיוג Build Number + latest)
   ├── 4. Deploy (פריסת קונטיינר)
   └── 5. Health Check (אימות מול /health)
          │
          ├── [הצלחה]: תיוג stable-backup + שליחת התראה לטלגרם ✅
          └── [כישלון]: הפעלת Rollback אוטומטי + התראת כשל לטלגרם ❌
          ▼
  [Docker Engine / Compose] ◄── [Ansible Configuration]
          │                           ├── inventory.ini
          ▼                           └── deploy.yml
  [קונטיינר האפליקציה: holiday-app]
    ├── האזנה פנימית: 3000
    └── מיפוי מארח: 8000
```
