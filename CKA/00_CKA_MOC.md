# 🗺️ CKA Map of Content (MOC)

> **Visual navigation guide for your CKA certification journey**

## 📊 Study Path Overview

```text
🎯 CKA CERTIFICATION JOURNEY
│
├── 📚 PREPARATION PHASE (Weeks 1-7)
│   │
│   ├── 🏗️ FOUNDATION (Weeks 1-2)
│   │   ├── Kubernetes Architecture (25%)
│   │   ├── Basic kubectl commands
│   │   └── Cluster components understanding
│   │
│   ├── ⚙️ WORKLOADS (Weeks 3-4)
│   │   ├── Pod lifecycle management
│   │   ├── Deployments & ReplicaSets
│   │   ├── Services & Networking (15%)
│   │   └── ConfigMaps & Secrets
│   │
│   ├── 🔧 ADVANCED TOPICS (Weeks 5-6)
│   │   ├── Storage & Volumes (10%)
│   │   ├── Network Policies
│   │   ├── Security & RBAC
│   │   └── Troubleshooting (30%) ⭐ HIGHEST WEIGHT
│   │
│   └── 🎓 EXAM PREP (Week 7)
│       ├── Practice Tests
│       ├── Time Management
│       └── Final Review
│
└── 🏆 EXAM DAY (Week 8)
    ├── Final checklist review
    ├── Environment setup
    └── Performance-based exam (2 hours)
```

## 🎯 Exam Domain Breakdown

### Visual Weight Distribution

```text
     CKA EXAM DOMAINS (Total: 100%)
     
🔧 Troubleshooting          ████████████████████████████████ 30%
🏗️ Cluster Architecture     ██████████████████████████ 25%
⚙️ Workloads & Scheduling   ███████████████ 15%
🌐 Services & Networking    ████████████████████ 20%
💾 Storage                  ██████████ 10%
```

## 📁 Content Organization Map

### 🗂️ Study Materials Structure

```text
CKA/
│
├── 📋 QUICK ACCESS FILES
│   ├── README.md                 ← 🚀 START HERE
│   ├── 00_MASTER_INDEX.md        ← 📚 Main study plan
│   ├── 00_Command_Reference.md   ← ⚡ Quick kubectl lookup
│   ├── 00_Labs_and_Practice.md   ← 💻 Hands-on scenarios
│   └── 99_EXAM_DAY_CHECKLIST.md  ← ✅ Final preparation
│
├── 🎯 NAVIGATION & PLANNING
│   ├── 00_CKA_MOC.md                     ← 🗺️ This visual guide
│   ├── 00_Study_Resources.md             ← 🔗 External links
│   ├── 00_Exam_Topics_Checklist.md       ← 📝 Progress tracking
│   └── 00_Spaced_Repetition_Template.md  ← 🧠 Memory system
│
└── 📖 STUDY CONTENT (by exam weight)
    ├── 5. Troubleshooting/ (30%) ⭐ PRIORITY #1
    ├── 1. Kubernetes Architecture/ (25%) ⭐ PRIORITY #2
    ├── 2. Workloads scheduling/ (15%)
    ├── 3. Services networking/ (20%)
    └── 4. Storage/ (10%)
```

## 🎯 Learning Path by Experience Level

### 🌱 New to Kubernetes

```text
Week 1-2: Foundation Building
├── Start → README.md (orientation)
├── Study → 1. Kubernetes Architecture/
├── Practice → Basic kubectl commands
└── Labs → 00_Labs_and_Practice.md (Beginner section)

Week 3-4: Core Concepts
├── Study → 2. Workloads scheduling/
├── Study → 3. Services networking/
├── Practice → 00_Command_Reference.md daily
└── Labs → Intermediate scenarios

Week 5-6: Advanced Topics
├── Study → 4. Storage/
├── Study → 5. Troubleshooting/ ⭐
├── Practice → Advanced labs
└── Simulate → Exam conditions

Week 7-8: Exam Preparation
├── Review → 00_MASTER_INDEX.md
├── Practice → Mock exams
├── Final → 99_EXAM_DAY_CHECKLIST.md
└── Exam → Performance test
```

### 🔧 Experienced with Kubernetes

```text
Week 1: Knowledge Gap Analysis
├── Review → 00_MASTER_INDEX.md (assess gaps)
├── Focus → Weak areas from self-assessment
└── Practice → 00_Labs_and_Practice.md (Advanced)

Week 2-3: Troubleshooting Focus (30% of exam!)
├── Deep dive → 5. Troubleshooting/
├── Practice → Complex scenarios
└── Time management → Exam simulations

Week 4: Final Preparation
├── Mock exams → Full-length practice tests
├── Command mastery → 00_Command_Reference.md
├── Final review → 99_EXAM_DAY_CHECKLIST.md
└── Exam ready → Performance test
```

## 📊 Topic Relationships & Dependencies

### 🔗 Content Flow Diagram

```text
                    KUBERNETES MASTERY PATH
                    
    🏗️ Architecture     →     ⚙️ Workloads     →     🌐 Services
         (Foundation)           (Core Skills)          (Networking)
              │                      │                      │
              └──────────────────────┼──────────────────────┘
                                     ↓
                             💾 Storage + 🔐 Security
                                     │
                                     ↓
                          🔧 Troubleshooting (Critical!)
                          
    Legend:
    → Sequential learning (recommended order)
    ↓ Builds upon previous knowledge
    🔧 Highest exam weight (30%)
```

### 📚 Cross-Topic Connections

| Topic | Connects To | Why Important |
|-------|------------|---------------|
| **Architecture** | All topics | Foundation for everything |
| **Workloads** | Services, Storage | Pods need networking & persistence |
| **Services** | Workloads, Networking | Expose and connect applications |
| **Storage** | Workloads | StatefulSets and data persistence |
| **Troubleshooting** | All topics | Debug everything above |

## 🎯 Study Session Templates

### 📝 Daily Study Session (1-2 hours)

```text
┌─ SESSION STRUCTURE ─┐
│ 1. Review (15 min)  │ ← Previous day concepts
│ 2. Study (45 min)   │ ← New topic from plan
│ 3. Practice (30 min)│ ← Hands-on labs
│ 4. Commands (15 min)│ ← kubectl reference
└─────────────────────┘
```

### 🔬 Practice Session (30-60 min)

```text
┌─ PRACTICE STRUCTURE ─┐
│ 1. Warm-up (10 min)  │ ← Basic kubectl commands
│ 2. Scenario (20 min) │ ← Lab from 00_Labs_and_Practice.md
│ 3. Debug (15 min)    │ ← Troubleshooting exercise
│ 4. Review (15 min)   │ ← Document learnings
└──────────────────────┘
```

### 🎯 Exam Prep Session (2 hours)

```text
┌─ EXAM PREP STRUCTURE ─┐
│ 1. Mock Exam (90 min) │ ← Timed full exam simulation
│ 2. Review (20 min)    │ ← Analyze mistakes
│ 3. Gaps (10 min)      │ ← Identify weak areas
└───────────────────────┘
```

## 🎯 Focus Areas by Week

### 📅 8-Week Study Calendar

```text
Week 1: 🏗️ FOUNDATION
├── Mon-Tue: Kubernetes Architecture basics
├── Wed-Thu: kubectl command mastery
├── Fri: Cluster setup understanding
└── Weekend: Basic pod management labs

Week 2: 🏗️ ARCHITECTURE DEEP DIVE
├── Mon-Tue: Control plane components
├── Wed-Thu: Worker node components
├── Fri: Networking basics
└── Weekend: Architecture troubleshooting

Week 3: ⚙️ WORKLOADS PART 1
├── Mon-Tue: Pods and ReplicaSets
├── Wed-Thu: Deployments and scaling
├── Fri: Rolling updates and rollbacks
└── Weekend: Workload labs

Week 4: ⚙️ WORKLOADS PART 2
├── Mon-Tue: ConfigMaps and Secrets
├── Wed-Thu: Resource management
├── Fri: Multi-container pods
└── Weekend: Advanced workload scenarios

Week 5: 🌐 SERVICES & 💾 STORAGE
├── Mon-Tue: Service types and networking
├── Wed-Thu: Persistent volumes and claims
├── Fri: Storage classes and CSI
└── Weekend: Service mesh basics

Week 6: 🔧 TROUBLESHOOTING FOCUS ⭐
├── Mon-Tue: Cluster troubleshooting
├── Wed-Thu: Node and pod debugging
├── Fri: Network troubleshooting
└── Weekend: Advanced debugging scenarios

Week 7: 🎓 EXAM PREPARATION
├── Mon-Tue: Mock exam #1
├── Wed-Thu: Mock exam #2
├── Fri: Final review and weak areas
└── Weekend: Relax and light review

Week 8: 🏆 EXAM WEEK
├── Mon-Tue: Light review only
├── Wed: Final checklist
├── Thu: Exam day prep
└── Fri: CKA EXAM! 🎉
```

## 🔗 Quick Navigation Links

### 📚 Essential Study Files

- **📖 Start Learning:** [Master Index](00_MASTER_INDEX.md)
- **⚡ Quick Commands:** [Command Reference](00_Command_Reference.md)
- **💻 Practice Labs:** [Labs & Practice](00_Labs_and_Practice.md)
- **🔗 External Resources:** [Study Resources](00_Study_Resources.md)

### 📝 Progress Tracking

- **✅ Detailed Checklist:** [Exam Topics Checklist](00_Exam_Topics_Checklist.md)
- **🧠 Memory System:** [Spaced Repetition](00_Spaced_Repetition_Template.md)
- **📊 Implementation Log:** [History](00_IMPLEMENTATION_HISTORY.md)

### 🏆 Exam Preparation

- **🎯 Final Prep:** [Exam Day Checklist](99_EXAM_DAY_CHECKLIST.md)
- **📋 Quick Reference:** [CKA Cheat Sheet](TEMPLATE_CKA_CHEAT_SHEET.md)

### 📖 Study Content (by priority)

1. **🔧 [Troubleshooting](5.%20Troubleshooting/)** (30% - Highest Priority!)
2. **🏗️ [Architecture](1.%20Kubernetes%20Architecture/)** (25% - Foundation)
3. **⚙️ [Workloads](2.%20Workloads%20scheduling/)** (15% - Core Skills)
4. **🌐 [Services](3.%20Services%20networking/)** (20% - Networking)
5. **💾 [Storage](4.%20Storage/)** (10% - Persistence)

## 💡 Success Tips

### 🎯 Study Strategy

- **Focus on troubleshooting** - It's 30% of the exam!
- **Practice daily kubectl** - Speed and muscle memory matter
- **Time yourself** - Exam pressure is real
- **Use official docs** - You can reference kubernetes.io during exam

### 🔧 Practical Tips

- **Set up aliases** - `alias k=kubectl` saves time
- **Master YAML generation** - `--dry-run=client -o yaml`
- **Practice vim/nano** - Exam environment text editors
- **Learn keyboard shortcuts** - Efficiency is key

### 🧠 Learning Tips

- **Understand concepts** - Don't just memorize commands
- **Practice scenarios** - Real-world troubleshooting
- **Review mistakes** - Learn from errors
- **Stay consistent** - Daily practice beats cramming

---

## 🎉 Motivation & Milestones

### 🏆 Certification Benefits

- **Career advancement** - Kubernetes expertise in high demand
- **Salary increase** - CKA certification shows practical skills
- **Industry recognition** - CNCF certification is globally respected
- **Practical knowledge** - Real-world Kubernetes administration skills

### 🎯 Study Milestones

- [ ] **Week 2:** Comfortable with basic kubectl commands
- [ ] **Week 4:** Can deploy and manage applications
- [ ] **Week 6:** Expert at troubleshooting common issues
- [ ] **Week 7:** Passing practice exams consistently
- [ ] **Week 8:** CKA CERTIFIED! 🎉

---

*Remember: The CKA exam tests practical skills, not theoretical knowledge. Focus on hands-on practice!*

---

*Last Updated: May 29, 2025*  
*CKA Exam Version: v1.30*  
*MOC Version: 1.0*
