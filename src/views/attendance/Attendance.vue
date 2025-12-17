<template>
  <div class="attendance-container" v-if="user">
    <h2>Attendance Records</h2>

    <div class="controls">
      <label>Select Class:</label>
      <select v-model="selectedClass" @change="fetchAttendance">
        <option value="">All Classes</option>
        <option v-for="cls in classes" :key="cls.id" :value="cls.id">
          {{ cls.label }}
        </option>
      </select>

      <label>Select Date:</label>
      <input type="date" v-model="selectedDate" @change="applyDate" />

      <span class="today-text">
        Applied Date: {{ activeDate || 'Not selected' }}
      </span>

      <input
        v-model="searchText"
        type="text"
        placeholder="Search by name or class..."
      />

      <div v-if="canEditAttendance" class="add-student-section">
        <label>Add Student:</label>
        <select v-model="studentToAdd">
          <option value="">Select Student</option>
          <option
            v-for="s in availableStudents"
            :key="s.id"
            :value="s.id"
          >
            {{ s.name }} ({{ s.classId }})
          </option>
        </select>
        <button @click="addAttendanceRecord">Add</button>
      </div>
    </div>

    <table v-if="filteredAttendance.length">
      <thead>
        <tr>
          <th>Date</th>
          <th>Photo</th>
          <th>ID Number</th>
          <th>Student Name</th>
          <th>Class</th>
          <th>Status</th>
          <th v-if="canEditAttendance">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="record in filteredAttendance"
          :key="record.date + record.id"
        >
          <td>{{ record.date }}</td>
          <td>
            <img
              v-if="record.photoUrl"
              :src="record.photoUrl"
              class="student-photo"
            />
            <span v-else>-</span>
          </td>
          <td>{{ record.idNumber || '-' }}</td>
          <td>{{ record.name || '-' }}</td>
          <td>{{ record.classId || '-' }}</td>
          <td>
            <div
              v-if="canEditAttendance && record.date === activeDate"
              class="status-group"
            >
              <label
                class="status-pill present"
                :class="{ active: attendance[record.id] === 'Present' }"
              >
                <input
                  type="radio"
                  :name="'status-' + record.id"
                  :checked="attendance[record.id] === 'Present'"
                  @change="setStatus(record.id, 'Present')"
                />
                Present
              </label>
              <label
                class="status-pill absent"
                :class="{ active: attendance[record.id] === 'Absent' }"
              >
                <input
                  type="radio"
                  :name="'status-' + record.id"
                  :checked="attendance[record.id] === 'Absent'"
                  @change="setStatus(record.id, 'Absent')"
                />
                Absent
              </label>
              <label
                class="status-pill weekoff"
                :class="{ active: attendance[record.id] === 'Week Off' }"
              >
                <input
                  type="radio"
                  :name="'status-' + record.id"
                  :checked="attendance[record.id] === 'Week Off'"
                  @change="setStatus(record.id, 'Week Off')"
                />
                Week Off
              </label>
            </div>
            <span
              v-else
              class="status-badge"
              :class="{
                present: record.status === 'Present',
                absent: record.status === 'Absent',
                weekoff: record.status === 'Week Off'
              }"
            >
              {{ record.status || '-' }}
            </span>
          </td>
          <td v-if="canEditAttendance">
            <button @click="editRecord(record.id)">Edit</button>
            <button @click="deleteRecord(record.id)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>

    <p v-else>No attendance records found.</p>
    <button v-if="canEditAttendance" @click="saveAttendance">
      Save Attendance
    </button>
  </div>

  <div v-else>
    <p>Loading or not logged in...</p>
  </div>
</template>

<script>
import { db } from "../../services/firebase"
import {
  collection,
  doc,
  getDocs,
  getDoc,
  query,
  where,
  setDoc
} from "firebase/firestore"

export default {
  data() {
    return {
      classes: Array.from({ length: 10 }, (_, i) => ({
        id: `class_${i + 1}`,
        label: `Class ${i + 1}`
      })),
      selectedClass: "",
      selectedDate: new Date().toISOString().substr(0, 10),
      activeDate: "",
      searchText: "",
      students: [],
      studentToAdd: "",
      attendance: {},
      allAttendanceRecords: [],
      user: JSON.parse(localStorage.getItem("user")) || {}
    }
  },
  computed: {
    canEditAttendance() {
      const role = (this.user.role || "").toString().toLowerCase()
      return ["teacher", "admin", "superadmin"].includes(role)
    },
    filteredAttendance() {
      const text = this.searchText.trim().toLowerCase()
      if (!text) return this.allAttendanceRecords
      return this.allAttendanceRecords.filter(
        r =>
          (r.name || "").toLowerCase().includes(text) ||
          (r.classId || "").toLowerCase().includes(text)
      )
    },
    availableStudents() {
      return this.students.filter(
        s =>
          !this.allAttendanceRecords.some(
            r => r.id === s.id && r.date === this.activeDate
          )
      )
    }
  },
  async mounted() {
    console.log("ATT USER =>", this.user)
    if (!this.user || !this.user.role) {
      alert("Please login first")
      this.$router.push("/login")
      return
    }
    this.activeDate = this.selectedDate
    await this.fetchAttendance()
  },
  methods: {
    // ✅ Helper: Safe data extraction
    safeData(data) {
      return {
        name: data.name || "",
        idNumber: data.idNumber || "",
        classId: data.classId || "",
        photoUrl: data.photoUrl || null
      }
    },

    applyDate() {
      if (!this.selectedDate) {
        alert("Please select a date first")
        return
      }
      this.activeDate = this.selectedDate
      this.fetchAttendance()
    },

    async fetchAttendance() {
      try {
        const studentsRef = collection(db, "students")
        let q = studentsRef

        console.log(
          "role:",
          this.user.role,
          "classId:",
          this.user.classId,
          "selectedClass:",
          this.selectedClass
        )

        if (this.user.role === "superadmin") {
          q = studentsRef
        } else if (this.user.role === "admin") {
          if (this.selectedClass) {
            q = query(studentsRef, where("classId", "==", this.selectedClass))
          } else {
            q = studentsRef
          }
        } else if (this.user.role === "teacher") {
          if (this.selectedClass) {
            q = query(studentsRef, where("classId", "==", this.selectedClass))
          } else if (this.user.classId) {
            q = query(studentsRef, where("classId", "==", this.user.classId))
          } else {
            q = studentsRef
          }
        }

        const snap = await getDocs(q)
        console.log("👀 Students fetched:", snap.size)
        
        // ✅ FIX 1: Safe student data mapping
        this.students = snap.docs.map(d => {
          const data = d.data()
          return { 
            id: d.id, 
            ...this.safeData(data),
            status: data.status || ""
          }
        })

        if (!this.activeDate) {
          this.allAttendanceRecords = []
          this.prepareAttendance()
          return
        }

        const docId = this.activeDate
        const dateRef = doc(db, "attendance", docId)
        const docSnap = await getDoc(dateRef)
        const attendanceData = docSnap.exists() ? docSnap.data() : {}

        const records = []

        for (const cid in attendanceData) {
          const classBlock = attendanceData[cid]
          if (!classBlock || typeof classBlock !== "object") continue

          if (this.selectedClass && cid !== this.selectedClass) continue

          for (const studentId in classBlock) {
            const rec = classBlock[studentId]
            records.push({
              date: this.activeDate,
              id: studentId,
              ...this.safeData(rec),  // ✅ Safe data
              status: rec.status || ""
            })
          }
        }

        this.allAttendanceRecords = records
        this.prepareAttendance()
      } catch (err) {
        console.error("Error fetching attendance:", err)
      }
    },

    prepareAttendance() {
      const selectedRecords = this.allAttendanceRecords.filter(
        r => r.date === this.activeDate
      )
      const map = {}
      selectedRecords.forEach(r => {
        if (r.status) map[r.id] = r.status
      })
      this.attendance = map
    },

    setStatus(studentId, status) {
      if (!this.canEditAttendance) return
      
      this.attendance = { ...this.attendance, [studentId]: status }
      
      const index = this.allAttendanceRecords.findIndex(
        r => r.id === studentId && r.date === this.activeDate
      )
      
      if (index !== -1) {
        this.$set(this.allAttendanceRecords, index, {
          ...this.allAttendanceRecords[index],
          status
        })
      } else {
        const student = this.students.find(s => s.id === studentId)
        if (!student) return
        
        // ✅ FIX 2: Safe new record creation
        this.allAttendanceRecords.push({
          date: this.activeDate,
          id: studentId,
          ...this.safeData(student),
          status
        })
      }
    },

    addAttendanceRecord() {
      if (!this.studentToAdd) {
        alert("Select a student")
        return
      }
      const student = this.students.find(s => s.id === this.studentToAdd)
      if (!student) return
      
      // ✅ FIX 3: Safe add record
      this.allAttendanceRecords.push({
        date: this.activeDate,
        id: student.id,
        ...this.safeData(student),
        status: "Present"
      })
      this.attendance[student.id] = "Present"
      this.studentToAdd = ""
    },

    editRecord(studentId) {
      const record = this.allAttendanceRecords.find(
        r => r.id === studentId && r.date === this.activeDate
      )
      if (!record) return
      
      const newStatus = prompt(
        "Enter status (Present / Absent / Week Off):",
        record.status
      )
      if (newStatus && ["Present", "Absent", "Week Off"].includes(newStatus)) {
        this.setStatus(studentId, newStatus)
      }
    },

    deleteRecord(studentId) {
      if (!confirm("Are you sure you want to delete this record?")) return
      
      // Remove from attendance map
      const { [studentId]: _, ...rest } = this.attendance
      this.attendance = rest
      
      // Remove from records
      this.allAttendanceRecords = this.allAttendanceRecords.filter(
        r => !(r.id === studentId && r.date === this.activeDate)
      )
    },

    // ✅ FIX 4: Bulletproof saveAttendance
    async saveAttendance() {
      try {
        if (!this.canEditAttendance) {
          alert("You cannot save attendance")
          return
        }
        if (!this.activeDate) {
          alert("First select date")
          return
        }

        // Only save records with status
        const recordsToSave = this.allAttendanceRecords.filter(s => s.status)
        
        if (recordsToSave.length === 0) {
          alert("No attendance marked to save")
          return
        }

        const docId = this.activeDate
        const dateRef = doc(db, "attendance", docId)
        const attendancePayload = {}

        recordsToSave.forEach(s => {
          const cid = s.classId || "unknown"
          if (!attendancePayload[cid]) {
            attendancePayload[cid] = {}
          }

          // 🔥 100% SAFE: No undefined values
          attendancePayload[cid][s.id] = {
            name: s.name || "",
            idNumber: s.idNumber || "",
            classId: s.classId || "",
            photoUrl: s.photoUrl || null,
            status: s.status
          }
        })

        console.log("📤 Saving:", recordsToSave.length, "records")
        console.log("Payload sample:", attendancePayload)
        
        await setDoc(dateRef, attendancePayload, { merge: true })
        
        alert(`✅ Attendance saved successfully for ${recordsToSave.length} students!`)
        await this.fetchAttendance()
        
      } catch (err) {
        console.error("💥 Save error:", err)
        alert("Failed to save: " + err.message)
      }
    }
  }
}
</script>

<style scoped>
.attendance-container{padding:2rem;background:#f5f5f5;min-height:100vh}
.controls{display:flex;flex-wrap:wrap;gap:1rem;margin-bottom:1rem;align-items:center}
.today-text{font-weight:bold;color:#2196F3}
.add-student-section{display:flex;align-items:center;gap:0.5rem}
table{width:100%;border-collapse:collapse;background:white;margin-top:1rem;box-shadow:0 2px 10px rgba(0,0,0,0.1)}
th,td{border:1px solid #ddd;padding:0.8rem;text-align:left}
th{background:#2196F3;color:white;font-weight:600}
.student-photo{width:35px;height:35px;border-radius:50%;object-fit:cover;border:2px solid #eee}
.status-group{display:flex;gap:0.4rem;flex-wrap:wrap}
.status-pill{display:inline-flex;align-items:center;gap:0.25rem;padding:0.4rem 0.8rem;border-radius:20px;font-size:0.85rem;font-weight:600;color:#fff;cursor:pointer;opacity:0.7;border:2px solid transparent;transition:all 0.2s}
.status-pill input{margin:0;accent-color:white}
.status-pill.present{background:#4caf50}
.status-pill.absent{background:#e53935}
.status-pill.weekoff{background:#ff9800}
.status-pill.active{opacity:1;transform:scale(1.05);border-color:rgba(255,255,255,0.5)}
.status-badge{padding:0.3rem 0.8rem;border-radius:20px;color:#fff;font-size:0.85rem;font-weight:500}
.status-badge.present{background:#4caf50}
.status-badge.absent{background:#e53935}
.status-badge.weekoff{background:#ff9800}
button{margin:0.2rem 0.1rem;padding:0.4rem 0.8rem;background:#1976d2;color:white;border:none;border-radius:5px;cursor:pointer;font-size:0.85rem;transition:background 0.2s}
button:hover{background:#0d47a1}
button:active{transform:scale(0.98)}
@media (max-width:768px) {
  .controls{flex-direction:column;align-items:stretch}
  .add-student-section{justify-content:center}
  .status-group{justify-content:center}
}
</style>

