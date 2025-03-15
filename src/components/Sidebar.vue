<template>
  <div class="sidebar">
    <h2>Projects</h2>
    <ul>
      <li v-for="project in projects" :key="project.name">
        {{ project.name }}
      </li>
    </ul>
    <button @click="addProject">Add New Project</button>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import fs from 'fs';
import path from 'path';

const projects = ref<{ name: string }[]>([]);

onMounted(() => {
  const workspacePath = fs.readFileSync(path.join(window.electronAPI.getPath('userData'), 'workspacePath.txt'), 'utf-8');
  const projectDirs = fs.readdirSync(workspacePath, { withFileTypes: true })
    .filter(dirent => dirent.isDirectory())
    .map(dirent => ({ name: dirent.name }));
  projects.value = projectDirs;
});

function addProject() {
  const projectName = prompt('Enter new project name:');
  if (projectName) {
    const workspacePath = fs.readFileSync(path.join(window.electronAPI.getPath('userData'), 'workspacePath.txt'), 'utf-8');
    const newProjectPath = path.join(workspacePath, projectName);
    if (!fs.existsSync(newProjectPath)) {
      fs.mkdirSync(newProjectPath);
      projects.value.push({ name: projectName });
    }
  }
}
</script>

<style scoped>
.sidebar {
  width: 250px;
  background-color: #f8f9fa;
  padding: 10px;
}

.sidebar h2 {
  font-size: 18px;
  margin-bottom: 10px;
}

.sidebar ul {
  list-style-type: none;
  padding: 0;
}

.sidebar li {
  margin-bottom: 5px;
}

.sidebar button {
  margin-top: 10px;
}
</style>
