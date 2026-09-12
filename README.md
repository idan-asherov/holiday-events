# 🚀 Jenkins CI/CD Pipeline & Ansible Infrastructure

**מערכת אירועי חברה (Holiday Events) - קו ייצור אוטומטי ותשתית כקוד (IaC)**

פרויקט זה מציג ארכיטקטורת CI/CD מודרנית המנהלת קו ייצור עבור אפליקציית Web (Node.js/Express). המערכת תוכננה בסטנדרטים של סביבת פיתוח וייצור מקומית (Local Production), עם דגש על אוטומציה מלאה ב-Jenkins, ניהול תצורה (Configuration Management) אידמפוטנטי בעזרת Ansible, ואסטרטגיית ניהול ענפים ב-Git.

---

## 📐 ארכיטקטורת המערכת (System Architecture)

המערכת פועלת במודל של הפרדת אחריות מלאה בין ה-Pipeline לבין מנוע הפריסה:

1. **אפליקציית הליבה (`holiday-events`):** שרת Backend המאזין פנימית על פורט 3000, מגיש ממשק משתמש סטטי, חושף API של אירועי חברה (`/api/events`), ומספק נתיב לבקרת בריאות (`/health`).
2. **שרת האוטומציה (`Jenkins`):** מנהל את מחזור החיים של הקוד - משיכה מ-GitHub, התקנת תלויות, בניית אימג' ב-Docker, פריסה ובדיקות בריאות.
3. **ניהול התצורה (`Ansible`):** מנוע אוטומציה עצמאי (`ansible/deploy.yml`) המנהל את מחזור החיים של הקונטיינר בצורה אידמפוטנטית מול מנוע ה-Docker Desktop המקומי.

```text
  [תחנת הפיתוח - ענפי Git]
                   │
           git push (main / feature)
                   ▼
       [מאגר מרוחק - GitHub SCM]
                   │
           משיכת קוד אוטומטית (SCM Poll / Build Now)
                   ▼
        [שרת האוטומציה - Jenkins Engine]
      ┌────────────────────────────────────────┐
      │ 1. Checkout (משיכת קוד המקור)           │
      │ 2. Dependencies (התקנת חבילות npm)      │
      │ 3. Build & Tag (בניית אימג' דוקר)      │
      │ 4. Local Deployment (פריסה מקומית)     │
      │ 5. Strict Health Check (בדיקת שפיות)   │
      └────────────────────────────────────────┘
                   │
         ניהול דרך Docker Socket
                   ▼
     [מנוע הקונטיינרים - Docker Desktop]  ◄── [Ansible Configuration Engine]
                   │                               ├── inventory.ini
                   ▼                               └── deploy.yml
       [קונטיינר האפליקציה: holiday-app]
         ├── פורט פנימי של השרת: 3000
         └── מיפוי לפורט חיצוני במחשב: 8000
```
