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
        v-for="column in scene.children"
        :key="column.id"
        class="column custom-scrollbar-container"
      >
        <div class="header">
          <div class="left">
            <span class="title">
              {{ column.name }}
            </span>

            <span class="count">
              {{ column.children.length }}
            </span>

            <button>
              <Icon name="Plus" :size="15" />
            </button>
          </div>

          <div class="right"></div>
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
          <Draggable v-for="card in column.children" :key="card.id">
            <div class="task">
              <p>{{ card.data }}</p>
            </div>
          </Draggable>
        </Container>
      </Draggable>
    </Container>
  </div>
</template>

<script>
import { Container, Draggable } from "vue3-smooth-dnd";
import { applyDrag, generateItems } from "../shared/utils/array";
import { Icon } from "lucide-vue-next";

const lorem = `Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. 
Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. 
Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.`;

const columnNames = ["To do", "In progress", "Complete", "Backlog", "Blocked"];

const scene = {
  type: "container",
  props: {
    orientation: "horizontal",
  },
  children: generateItems(4, (i) => ({
    id: `column${i}`,
    type: "container",
    name: columnNames[i],
    props: {
      orientation: "vertical",
      className: "card-container",
    },
    children: generateItems(+(Math.random() * 10).toFixed() + 5, (j) => ({
      type: "draggable",
      id: `${i}${j}`,
      data: lorem.slice(0, Math.floor(Math.random() * 150) + 30),
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
      scene.children = applyDrag(scene.children, dropResult);
      this.scene = scene;
    },

    onCardDrop(columnId, dropResult) {
      if (dropResult.removedIndex !== null || dropResult.addedIndex !== null) {
        const scene = Object.assign({}, this.scene);
        const column = scene.children.filter((p) => p.id === columnId)[0];
        const columnIndex = scene.children.indexOf(column);

        const newColumn = Object.assign({}, column);
        newColumn.children = applyDrag(newColumn.children, dropResult);
        scene.children.splice(columnIndex, 1, newColumn);

        this.scene = scene;
      }
    },

    getCardPayload(columnId) {
      return (index) => {
        return this.scene.children.filter((p) => p.id === columnId)[0].children[
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
            @apply size-10;
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
