# Security Specification: 나의 AI 프롬프트 창고

## 1. Data Invariants
- Every document in `/prompts/{promptId}` must have a valid `promptId` matching `^[a-zA-Z0-9_\-]+$` (length <= 128).
- Every prompt must contain exactly the four fields `title`, `category`, `content`, and `createdAt` (no shadow/ghost fields).
- `title` must be a non-empty string between 1 and 100 characters.
- `category` must be one of the three allowed categories: `'사진 합성'`, `'스타일 변환'`, `'캐릭터·포스터'`.
- `content` must be a non-empty string between 1 and 10,000 characters (supporting long prompts over 1,000 characters).
- `createdAt` must equal `request.time` on creation and remain immutable across updates.
- Read (`get`, `list`) on `/prompts/{promptId}` is open to anyone (no login required).
- Write (`create`, `update`, `delete`) on `/prompts/{promptId}` is restricted strictly to verified Google login `yuwww4952@gmail.com`.
- All other collections and documents are completely blocked from both read and write.

## 2. Adversarial "Dirty Dozen" Payloads (Expected: PERMISSION_DENIED)
1. **Unauthenticated Create**: Guest calls `addDoc(collection(db, "prompts"), ...)`
2. **Non-Admin Authenticated Delete**: Non-admin user calls `deleteDoc(...)`
3. **Unverified Admin Email Spoof**: Token has `email: "yuwww4952@gmail.com"` with `email_verified: false`
4. **Shadow Field Injection**: `{ title: "A", category: "사진 합성", content: "B", createdAt: serverTimestamp(), isAdmin: true }`
5. **Oversized Document ID**: `doc(db, "prompts", "a".repeat(200))`
6. **Oversized Title (>100 chars)**: `{ title: "x".repeat(101), category: "사진 합성", content: "Valid", createdAt: serverTimestamp() }`
7. **Unlisted Category**: `{ title: "Test", category: "임의 분류", content: "Valid", createdAt: serverTimestamp() }`
8. **Oversized Content (>10,000 chars)**: `{ title: "Test", category: "스타일 변환", content: "x".repeat(10001), createdAt: serverTimestamp() }`
9. **Wrong Content Type**: `{ title: "Test", category: "스타일 변환", content: 12345, createdAt: serverTimestamp() }`
10. **Forged Past Timestamp on Create**: `{ title: "Test", category: "캐릭터·포스터", content: "Valid", createdAt: Timestamp.fromDate(new Date("2020-01-01")) }`
11. **Mutating `createdAt` on Update**: `updateDoc(ref, { createdAt: serverTimestamp() })`
12. **Read or Write to Any Other Collection**: `getDoc(doc(db, "users", "u1"))` or `setDoc(doc(db, "settings", "s1"), { a: 1 })`
