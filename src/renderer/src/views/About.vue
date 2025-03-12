<template>
  <div class="about">
    <el-card class="about-card">
      <template #header>
        <div class="card-header">
          <el-icon><InfoFilled /></el-icon>
          <span>关于</span>
        </div>
      </template>
      <el-descriptions title="项目信息" :column="1" border>
        <el-descriptions-item label="项目名称">Electron + Vue3 应用</el-descriptions-item>
        <el-descriptions-item label="技术栈">
          <el-tag>Electron</el-tag>
          <el-tag type="success">Vue3</el-tag>
          <el-tag type="warning">Vite</el-tag>
          <el-tag type="danger">Element Plus</el-tag>
        </el-descriptions-item>
        <el-descriptions-item label="项目描述">
          这是一个使用 Electron + Vue3 + Element Plus 构建的现代桌面应用
        </el-descriptions-item>
      </el-descriptions>

      <div class="actions">
        <router-link to="/">
          <el-button type="primary">
            <el-icon><Back /></el-icon>
            返回首页
          </el-button>
        </router-link>
      </div>
    </el-card>
  </div>
</template>

<script setup></script>

<style scoped>
.about {
  padding: 20px;
}

.about-card {
  max-width: 800px;
  margin: 0 auto;
}

.card-header {
  display: flex;
  align-items: center;
  gap: 8px;
}

.el-tag {
  margin-right: 8px;
}

.actions {
  margin-top: 20px;
  text-align: center;
}
</style>
