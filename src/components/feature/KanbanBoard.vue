<template>
  <div class="kanban-container custom-scrollbar-container">
    <Container
      orientation="horizontal"
      @drop="onColumnDrop($event)"
      drag-handle-selector=".column-drag-handle"
      @drag-start="dragStart"
      :drop-placeholder="upperDropPlaceholderOptions"
      class="kanban custom-scrollbar"
    >
      <Draggable
        v-for="column in scene.columns"
        :key="column.id"
        class="column custom-scrollbar-container"
      >
        <div class="header">
          <div class="left">
            <span class="title">
              {{ column.name }}
            </span>

            <span class="count">
              {{ column.tasks.length }}
            </span>
          </div>

          <div class="right">
            <!-- Add New Task -->
            <button @click="addNewTask(column.id)">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="24"
                height="24"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
                class="lucide lucide-plus"
              >
                <path d="M5 12h14" />
                <path d="M12 5v14" />
              </svg>
            </button>
          </div>
        </div>

        <Container
          group-name="col"
          @drop="(e) => onCardDrop(column.id, e)"
          @drag-start="(e) => log('drag start', e)"
          @drag-end="(e) => log('drag end', e)"
          :get-child-payload="getCardPayload(column.id)"
          drag-class="task-ghost"
          drop-class="card-ghost-drop"
          :drop-placeholder="dropPlaceholderOptions"
          class="task-list custom-scrollbar"
        >
          <Draggable v-for="card in column.tasks" :key="card.id">
            <div class="task">
              <p>{{ card.title }}</p>
            </div>
          </Draggable>
        </Container>
      </Draggable>
    </Container>
  </div>
</template>

<script>
import { Container, Draggable } from "vue3-smooth-dnd";
import { applyDrag, generateItems } from "../../shared/utils/array";
import { v4 as uuidv4 } from "uuid";

const lorem = `Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. 
Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. 
Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.`;

const columnNames = ["To do", "In progress", "Complete", "Backlog", "Blocked"];

const scene = {
  type: "container",
  props: {
    orientation: "horizontal",
  },
  columns: generateItems(4, (i) => ({
    id: uuidv4(),
    type: "container",
    name: columnNames[i],
    props: {
      orientation: "vertical",
      className: "card-container",
    },
    tasks: generateItems(+(Math.random() * 10).toFixed() + 5, (j) => ({
      type: "draggable",
      id: uuidv4(),
      title: lorem.slice(0, Math.floor(Math.random() * 150) + 30),
    })),
  })),
};

export default {
  components: { Container, Draggable },

  data() {
    return {
      scene,
      upperDropPlaceholderOptions: {
        className: "cards-drop-preview",
        animationDuration: "150",
        showOnTop: true,
      },
      dropPlaceholderOptions: {
        className: "drop-preview",
        animationDuration: "150",
        showOnTop: true,
      },
    };
  },

  methods: {
    onColumnDrop(dropResult) {
      const scene = Object.assign({}, this.scene);
      scene.columns = applyDrag(scene.columns, dropResult);
      this.scene = scene;
    },

    onCardDrop(columnId, dropResult) {
      if (dropResult.removedIndex !== null || dropResult.addedIndex !== null) {
        const scene = Object.assign({}, this.scene);
        const column = scene.columns.filter((p) => p.id === columnId)[0];
        const columnIndex = scene.columns.indexOf(column);

        const newColumn = Object.assign({}, column);
        newColumn.tasks = applyDrag(newColumn.tasks, dropResult);
        scene.columns.splice(columnIndex, 1, newColumn);

        this.scene = scene;
      }
    },

    getCardPayload(columnId) {
      return (index) => {
        return this.scene.columns.filter((p) => p.id === columnId)[0].tasks[
          index
        ];
      };
    },

    dragStart() {
      console.log("drag started");
    },

    log(...params) {
      console.log(...params);
    },

    addNewTask(columnId) {
      const scene = Object.assign({}, this.scene);
      const column = scene.columns.find((p) => p.id === columnId);
      if (column) {
        const totalTaskCount = column.tasks.length;

        column.tasks.unshift({
          type: "draggable",
          id: uuidv4(),
          title: "New Task",
        });
        this.scene = scene;
      }
    },
  },
};
</script>

<style scoped lang="scss">
.kanban-container {
  @apply flex flex-col p-5 mr-4 h-full;

  .kanban {
    @apply flex flex-row gap-5;

    .column {
      @apply flex flex-col gap-4 min-w-[350px] p-4 rounded-lg
        bg-gradient-to-b from-zinc-800 to-zinc-900;

      .header {
        @apply flex flex-row gap-3 justify-between items-center;

        .left {
          @apply flex flex-row gap-2 items-center;

          .title {
            @apply text-white;
          }

          .count {
            @apply text-gray-500 text-sm;
          }
        }

        .right {
          @apply flex flex-row gap-2 justify-end;

          button {
            @apply p-1.5 rounded-sm bg-zinc-800
              transition-all duration-200 ease-in-out;

            &:hover {
              @apply bg-zinc-700/80;
            }

            svg {
              @apply size-4;
            }
          }
        }
      }

      .task-list {
        @apply flex flex-col gap-2 pr-3 h-full overflow-y-auto;

        &.task-ghost {
          @apply bg-zinc-400 border-zinc-700;
        }

        .task {
          @apply flex flex-col gap-2 border py-4 px-3 rounded-lg
            min-h-[100px]
            bg-zinc-800 border-zinc-700 hover:border-zinc-600
            transition-all duration-300 ease-in-out cursor-pointer;
        }
      }
    }
  }
}

.task-list .smooth-dnd-draggable-wrapper {
  @apply overflow-visible #{!important};
}
</style>
