<template>
  <div class="home">
    <el-card class="welcome-card">
      <template #header>
        <div class="card-header">
          <el-icon><HomeFilled /></el-icon>
          <span>欢迎使用</span>
        </div>
      </template>
      <el-row class="mb-4" justify="center">
        <el-col :span="12">
          <el-result
            icon="success"
            title="Electron + Vue3 + Element Plus"
            sub-title="您的应用已经成功集成了所有必要的组件"
          >
            <template #extra>
              <router-link to="/about">
                <el-button type="primary">了解更多</el-button>
              </router-link>
            </template>
          </el-result>
        </el-col>
      </el-row>
    </el-card>
  </div>
</template>

<script setup></script>

<style scoped>
.home {
  padding: 20px;
}

.welcome-card {
  max-width: 800px;
  margin: 0 auto;
}

.card-header {
  display: flex;
  align-items: center;
  gap: 8px;
}

.mb-4 {
  margin-bottom: 1rem;
}
</style>
