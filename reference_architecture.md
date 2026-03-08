# 🧭 Reference Production Architecture (Control-Aware)

---

## Target Profile

| Attribute | Value |
|-----------|-------|
| Workload | Hybrid (read-heavy API + write transactions) |
| Traffic shape | Spiky + diurnal |
| Initial scale | 5k QPS peak |
| 12-month projection | 40k QPS |
| Worst-case spike | ×8 |
| Consistency | Hybrid |
| Region | Single-primary, multi-AZ |

---

# 🏗 High-Level Architecture
