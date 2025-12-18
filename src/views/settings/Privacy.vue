<template>
  <div class="privacy-page">
    <!-- Header -->
    <div class="privacy-header">
      <h1>Privacy & Data Safety</h1>
      <p>
        We are committed to protecting the privacy and security of all school,
        student, parent, and staff data.
      </p>
    </div>

    <!-- Content Cards -->
    <div class="privacy-cards">
      <div class="privacy-card">
        <h3>1. Data Ownership</h3>
        <p>
          All data stored in the school application (students, parents, teachers)
          is fully owned by the school. The developer or any third party does not
          claim ownership or misuse this data.
        </p>
      </div>

      <div class="privacy-card">
        <h3>2. Data Collection</h3>
        <ul>
          <li>Student name, class, and roll number</li>
          <li>Parent contact details</li>
          <li>Attendance and academic records</li>
        </ul>
        <p>
          We do not collect any unnecessary or sensitive personal information.
        </p>
      </div>

      <div class="privacy-card">
        <h3>3. Data Usage</h3>
        <p>
          The collected data is used only for school management purposes such as
          attendance, notices, communication, and academic administration.
          Data is never used for advertising or marketing.
        </p>
      </div>

      <div class="privacy-card">
        <h3>4. Data Security</h3>
        <ul>
          <li>Secure login using user ID and password</li>
          <li>Encrypted data storage</li>
          <li>Role-based access control</li>
          <li>Secure cloud-based servers</li>
        </ul>
      </div>

      <div class="privacy-card">
        <h3>5. Access Control</h3>
        <ul>
          <li><strong>Admin:</strong> Full access to all data</li>
          <li><strong>Teacher:</strong> Access only to assigned classes</li>
          <li><strong>Parent:</strong> Access only to their child’s data</li>
        </ul>
      </div>

      <div class="privacy-card">
        <h3>6. Data Backup</h3>
        <p>
          Regular automated backups are performed to ensure data safety and
          prevent accidental data loss.
        </p>
      </div>

      <div class="privacy-card">
        <h3>7. Data Deletion & Export</h3>
        <p>
          Upon official request by the school, all data can be securely exported
          or permanently deleted from the system.
        </p>
      </div>

      <div class="privacy-card">
        <h3>8. App Permissions</h3>
        <p>
          The application requests only essential permissions. It does not access
          camera, storage, or location without a valid purpose.
        </p>
      </div>

      <div class="privacy-card">
        <h3>9. Third-Party Sharing</h3>
        <p>
          School data is never shared with third parties, except when required
          for core system functionality or by applicable law.
        </p>
      </div>

      <div class="privacy-card highlight">
        <h3>10. Contact & Support</h3>
        <p>
          For any privacy or data-related questions, the school administrator
          may contact the application support or developer.
        </p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "PrivacySettings"
}
</script>

<style scoped>
.privacy-page {
  background: #f4f6f8;
  min-height: 100vh;
  padding: 32px 16px;
}

.privacy-header {
  max-width: 900px;
  margin: 0 auto 32px;
  text-align: center;
}

.privacy-header h1 {
  font-size: 2rem;
  margin-bottom: 8px;
  color: #1f2937;
}

.privacy-header p {
  color: #6b7280;
  font-size: 1rem;
}

.privacy-cards {
  max-width: 900px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: 1fr;
  gap: 18px;
}

.privacy-card {
  background: #ffffff;
  border-radius: 12px;
  padding: 20px 22px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.privacy-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 18px rgba(0, 0, 0, 0.08);
}

.privacy-card h3 {
  margin-bottom: 8px;
  color: #1976d2;
  font-size: 1.1rem;
}

.privacy-card p {
  color: #374151;
  line-height: 1.7;
}

.privacy-card ul {
  margin: 8px 0 6px 18px;
  color: #374151;
}

.privacy-card li {
  margin-bottom: 4px;
}

.privacy-card.highlight {
  border-left: 5px solid #1976d2;
  background: #f8fbff;
}

/* Responsive */
@media (max-width: 600px) {
  .privacy-header h1 {
    font-size: 1.6rem;
  }
}
</style>

