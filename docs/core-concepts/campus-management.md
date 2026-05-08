# Campus Management Deep-Dive

This document covers the technical architecture of the multi-campus system — how campus context is determined, how data is partitioned, and how the campus switcher operates at the code level.

---

## Campus Context Lifecycle

Every page load follows this sequence:

