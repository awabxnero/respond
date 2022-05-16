

<template>
  <div class="container">
    <div class="progress">
      <p>Progress</p>
      <div class="progress-bar">
        <div
          class="progress-done"
          :style="{ width: (100 * tasksDone) / totalTasks + '%' }"
        >
          <p class="progress-anchor">.</p>
        </div>
        <p class="progress-text">
          Done {{ tasksDone }} out of {{ totalTasks }}
        </p>
      </div>
    </div>
    <div class="buttons">
      <input
        :value="text"
        @input="(event) => search(event.target.value)"
        placeholder="Search"
        class="search-bar"
      />

      <v-dialog v-model="dialog" width="unset">
        <template v-slot:activator="{ on, attrs }">
          <div
            @click="dialog = true"
            class="black-button"
            dark
            v-bind="attrs"
            v-on="on"
          >
            Add Task
          </div>
        </template>

        <v-card>
          <div class="task-form">
            <v-text-field
              class="text-field"
              label="Title"
              v-model="title"
            ></v-text-field>
            <v-textarea label="Description" v-model="description"></v-textarea>
            <v-text-field
              label="Estimated time"
              v-model="estimation"
            ></v-text-field>
            <p>Upload attachments</p>
            <input
              type="file"
              id="file"
              ref="myFiles"
              class="custom-file-input"
              @change="uploadAttachment"
              multiple
            />
            <div class="attachments">
              <div
                :key="attachment.name"
                v-for="attachment in createAttachments"
                class="file"
              >
                <div class="attachment-name">
                  <p>
                    {{ attachment.name }}
                  </p>
                </div>
                <p
                  @click="removeAttachment(attachment.name)"
                  class="file-remove"
                >
                  X
                </p>
              </div>
            </div>
            <div class="form-label-container">
              <div class="form-labels">
                <v-text-field
                  label="Label"
                  height="100%"
                  v-model="labelName"
                ></v-text-field>
                <v-select
                  :items="labelColors"
                  label="Color"
                  v-model="labelColor"
                ></v-select>
              </div>
              <button class="black-button add-label" @click="createLabel">
                Add label
              </button>
              <div class="label-preview-container">
                <div
                  :key="label.name"
                  v-for="label in createLabels"
                  class="label-preview"
                  :style="{ backgroundColor: label.color }"
                >
                  <p>
                    {{ label.name }}
                  </p>
                  <p @click="removeLabel(label.name)" class="file-remove">X</p>
                </div>
              </div>
            </div>
          </div>

          <v-divider></v-divider>

          <v-card-actions>
            <v-spacer></v-spacer>
            <div
              v-if="!dialogEditMode"
              class="black-button"
              text
              @click="createTask"
            >
              Create Task
            </div>
            <div
              v-if="dialogEditMode"
              class="black-button"
              text
              @click="editTask"
            >
              Edit Task
            </div>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </div>
    <div class="board">
      <div
        class="column drop-zone"
        @drop="onDrop($event, 'pending')"
        @dragover.prevent
        @dragenter.prevent
      >
        <div class="column-header">
          <p>Pending</p>
        </div>

        <div
          class="task"
          v-for="item in pendingTasks"
          :key="item.title"
          draggable
          @dragstart="startDrag($event, item)"
          :style="{ backgroundColor: item?.searched }"
        >
          <div class="card-content">
            <button v-on:click="openEditDialog(item)" class="edit-button">
              Edit
              <v-icon large class="edit-icon" color="white">
                mdi-pencil
              </v-icon>
            </button>
            <div>
              <p class="card-item-title">Tasks:</p>
              <p>
                {{ item?.title }}
              </p>
            </div>
            <div>
              <p class="card-item-title">Description:</p>
              <p>
                {{ item?.description }}
              </p>
            </div>

            <div>
              <p class="card-item-title">Estimated Time:</p>
              <p>
                {{ item?.estimation }}
              </p>
            </div>
            <div>
              <p class="card-item-title">Labels:</p>

              <div class="labels">
                <p
                  class="black-button"
                  :style="{
                    backgroundColor: label.color,
                    marginRight: '5px',
                    padding: '8px',
                  }"
                  v-for="label in item?.labels"
                  :key="label.name"
                >
                  {{ label.name }}
                </p>
              </div>
            </div>

            <div v-if="item?.attachments?.length > 0">
              <p class="card-item-title">Attachments:</p>
              <p>
                {{ item?.attachments?.length }}
              </p>
              <button
                class="black-button"
                style="margin-top: 10px"
                @click="toggleAttachmentsPreview(item)"
              >
                View Attachments
              </button>
            </div>

            <div v-if="item?.preview" class="attacments-preview">
              <div
                :key="attachment.name"
                v-for="attachment in item?.attachments"
                class="file"
              >
                {{ attachment.name }}
              </div>
            </div>
          </div>
        </div>
      </div>

      <div
        class="column drop-zone"
        @drop="onDrop($event, 'processing')"
        @dragover.prevent
        @dragenter.prevent
      >
        <div class="column-header">
          <p>Processing</p>
        </div>
        <div
          class="task"
          v-for="item in processingTasks"
          :key="item.title"
          draggable
          @dragstart="startDrag($event, item)"
          :style="{ backgroundColor: item?.searched }"
        >
          <div class="card-content">
            <button v-on:click="openEditDialog(item)" class="edit-button">
              Edit
              <v-icon large class="edit-icon" color="white">
                mdi-pencil
              </v-icon>
            </button>
            <div>
              <p class="card-item-title">Tasks:</p>
              <p>
                {{ item?.title }}
              </p>
            </div>
            <div>
              <p class="card-item-title">Description:</p>
              <p>
                {{ item?.description }}
              </p>
            </div>

            <div>
              <p class="card-item-title">Estimated Time:</p>
              <p>
                {{ item?.estimation }}
              </p>
            </div>
            <div>
              <p class="card-item-title">Labels:</p>

              <div class="labels">
                <p
                  class="black-button"
                  :style="{
                    backgroundColor: label.color,
                    marginRight: '5px',
                    padding: '8px',
                  }"
                  v-for="label in item?.labels"
                  :key="label.name"
                >
                  {{ label.name }}
                </p>
              </div>
            </div>

            <div v-if="item?.attachments?.length > 0">
              <p class="card-item-title">Attachments:</p>
              <p>
                {{ item?.attachments?.length }}
              </p>
              <button
                class="black-button"
                style="margin-top: 10px"
                @click="toggleAttachmentsPreview(item)"
              >
                View Attachments
              </button>
            </div>

            <div v-if="item?.preview" class="attacments-preview">
              <div
                :key="attachment.name"
                v-for="attachment in item?.attachments"
                class="file"
              >
                {{ attachment.name }}
              </div>
            </div>
          </div>
        </div>
      </div>

      <div
        class="column drop-zone"
        @drop="onDrop($event, 'done')"
        @dragover.prevent
        @dragenter.prevent
      >
        <div class="column-header">
          <p>Done</p>
        </div>
        <div
          class="task"
          v-for="item in doneTasks"
          :key="item.title"
          draggable
          @dragstart="startDrag($event, item)"
          :style="{ backgroundColor: item?.searched }"
        >
          <div class="card-content">
            <button v-on:click="openEditDialog(item)" class="edit-button">
              Edit
              <v-icon large class="edit-icon" color="white">
                mdi-pencil
              </v-icon>
            </button>
            <div>
              <p class="card-item-title">Tasks:</p>
              <p>
                {{ item?.title }}
              </p>
            </div>
            <div>
              <p class="card-item-title">Description:</p>
              <p>
                {{ item?.description }}
              </p>
            </div>

            <div>
              <p class="card-item-title">Estimated Time:</p>
              <p>
                {{ item?.estimation }}
              </p>
            </div>
            <div>
              <p class="card-item-title">Labels:</p>

              <div class="labels">
                <p
                  class="black-button"
                  :style="{
                    backgroundColor: label.color,
                    marginRight: '5px',
                    padding: '8px',
                  }"
                  v-for="label in item?.labels"
                  :key="label.name"
                >
                  {{ label.name }}
                </p>
              </div>
            </div>

            <div v-if="item?.attachments?.length > 0">
              <p class="card-item-title">Attachments:</p>
              <p>
                {{ item?.attachments?.length }}
              </p>
              <button
                class="black-button"
                style="margin-top: 10px"
                @click="toggleAttachmentsPreview(item)"
              >
                View Attachments
              </button>
            </div>

            <div v-if="item?.preview" class="attacments-preview">
              <div
                :key="attachment.name"
                v-for="attachment in item?.attachments"
                class="file"
              >
                {{ attachment.name }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    const tasksList = [
      {
        order: 0,
        title: "Item A",
        description:
          "Lorem Ipsum is simply dummy text, Lorem Ipsum is simply dummy text,Lorem Ipsum is simply dummy text.",
        labels: [
          {
            name: "label1",
            color: "red",
          },
          {
            name: "label2",
            color: "green",
          },
        ],
        attachments: [
          {
            name: "file1",
          },
          {
            name: "file2",
          },
          {
            name: "file3",
          },
        ],
        estimation: "1h",
        status: "pending",
        preview: false,
        searched: false,
      },
      {
        order: 1,
        title: "Item B",
        description:
          "Lorem Ipsum is simply dummy text, Lorem Ipsum is simply dummy text,Lorem Ipsum is simply dummy text.",
        labels: [
          {
            name: "label1",
            color: "red",
          },
          {
            name: "label2",
            color: "green",
          },
        ],
        attachments: [
          {
            name: "file1",
          },
          {
            name: "file2",
          },
          {
            name: "file3",
          },
        ],
        estimation: "1h",
        status: "processing",
        preview: false,
        searched: false,
      },
      {
        order: 2,
        title: "Item C",
        description:
          "Lorem Ipsum is simply dummy text, Lorem Ipsum is simply dummy text,Lorem Ipsum is simply dummy text.",
        labels: [
          {
            name: "label1",
            color: "red",
          },
          {
            name: "label2",
            color: "green",
          },
        ],
        attachments: [
          {
            name: "file1",
          },
          {
            name: "file2",
          },
          {
            name: "file3",
          },
        ],
        estimation: "1h",
        status: "done",
        preview: false,
        searched: false,
      },
    ];
    return {
      dialog: false,
      dialogEditMode: false,
      editingTask: null,
      createAttachments: [],
      labelColors: ["green", "red", "yellow"],
      createLabels: [],
      tasks: tasksList.sort((a, b) => (a.order > b.order ? -1 : 1)),
      draggedTask: null,
    };
  },
  computed: {
    totalTasks() {
      return this.tasks.filter(
        (task) => task.status === "pending" || "processing"
      )?.length;
    },
    tasksDone() {
      return this.tasks.filter((task) => task.status === "done")?.length;
    },
    pendingTasks() {
      return this.tasks.filter((item) => item.status === "pending");
    },
    processingTasks() {
      return this.tasks.filter((item) => item.status === "processing");
    },
    doneTasks() {
      return this.tasks.filter((item) => item.status === "done");
    },
  },
  methods: {
    startDrag(evt, item) {
      console.log(this.tasks);

      this.draggedTask = item.order;
      evt.dataTransfer.dropEffect = "move";
      evt.dataTransfer.effectAllowed = "move";
      evt.dataTransfer.setData("itemID", item.order);
    },
    onDrop(evt, status) {
      const itemID = evt.dataTransfer.getData("itemID");
      const item = this.tasks.find((_item) => _item.order == itemID);
      item.status = status;
      this.draggedTask = null;
    },

    createTask() {
      this.tasks.push({
        order: this.tasks.length + 1,
        title: this.title,
        description: this.description,
        labels: this.createLabels,
        attachments: this.createAttachments,
        estimation: this.estimation,
        status: "pending",
        preview: false,
        searched: false,
      });
      this.dialog = false;
    },
    uploadAttachment(event) {
      this.createAttachments = [
        ...event.target.files,
        ...this.createAttachments,
      ];
    },
    removeAttachment(fileName) {
      this.createAttachments = this.createAttachments.filter(
        (file) => file.name !== fileName
      );
    },
    toggleAttachmentsPreview(data) {
      const item = this.tasks.find((_item) => _item.order == data.order);
      item.preview = !item.preview;
      this.draggedTask = null;
    },
    createLabel(item) {
      this.createLabels.push({
        name: this.labelName,
        color: this.labelColor,
      });
      this.labelName = "";
      this.labelColor = "";
    },
    removeLabel(labelName) {
      this.createLabels = this.createLabels.filter(
        (label) => label.name !== labelName
      );
    },
    openEditDialog(item) {
      this.dialog = true;
      this.dialogEditMode = true;
      this.title = item.title;
      this.description = item.description;
      this.createLabels = item.labels;
      this.createAttachments = item.attachments;
      this.estimation = item.estimation;
      this.editingTask = item.order;
    },
    editTask() {
      const task = this.tasks.find((_item) => _item.order == this.editingTask);
      task.title = this.title;
      task.description = this.description;
      task.labels = this.createLabels;
      task.attachments = this.createAttachments;
      this.dialog = false;
    },
    search(text) {
      const searchResults = this.tasks.filter((item) => {
        return item.title.toLowerCase().includes(text.toLowerCase());
      });
      searchResults.forEach((item) => {
        item.searched = "#E53170";
      });
    },
  },
};
</script>
<style>
.container {
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  width: 100%;
  min-height: 100vh;
  height: 100%;

  overflow: hidden;
  background: #fafafa;
}
.buttons {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  width: 100%;
  height: 8vh;
  padding: 10px;
  border-bottom: 2px solid #272343;
  padding-right: 20px;
  padding-left: 20px;
}
.search-bar {
  width: 20%;
  border: 2px solid #272343;
  padding: 5px;
}
.board {
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  width: 100%;
  height: 77vh;
}
.task-form {
  width: 50vw;
  padding: 40px;
}
.progress {
  height: 15vh;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  background: #272343;
  color: white;
  font-weight: bolder;
}
.column {
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;
  flex-grow: 2;
  align-items: center;
  margin-left: 20px;
  margin-right: 20px;
  background: #ffe9ed !important;
  border-right: 2px solid #272343;
  border-left: 2px solid #272343;
  overflow: scroll;
}
.column::-webkit-scrollbar {
  width: 0;
  background: transparent;
}
.column::-webkit-scrollbar-thumb {
  background: #ff0000;
}
.column-header {
  width: 100%;
  padding: 10px;
  background: #272343;
  display: flex;
  flex-direction: column;
  align-items: center;
  color: white;
  font-weight: bolder;
}
.task,
::selection {
  color: none;
  background: none;
}
.task {
  font-size: smaller;
  user-select: all;
  touch-action: manipulation;
  cursor: grab;
  margin-bottom: 5px;
  display: flex;
  flex-direction: column;
  width: 90%;
  padding: 20px;
  margin-top: 40px;
  margin-bottom: 20px;
  padding-bottom: 40px;
  padding-top: 40px;
  --tw-shadow: -5px 5px 0 #272343;
  --tw-shadow-colored: -5px 5px 0 var(--tw-shadow-color);
  box-shadow: var(--tw-ring-offset-shadow, 0 0 #0000),
    var(--tw-ring-shadow, 0 0 #0000), var(--tw-shadow);
  --tw-shadow-color: #272343;
  --tw-shadow: var(--tw-shadow-colored);
  transition-property: box-shadow;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 200ms;
  --tw-bg-opacity: 1;
  background-color: rgb(255 255 255 / var(--tw-bg-opacity));
  --tw-text-opacity: 1;
  color: rgb(0 0 0 / var(--tw-text-opacity));
  border-width: 2px;
  border-style: solid;
  --tw-border-opacity: 1;
  border-color: rgb(0 0 0 / var(--tw-border-opacity));
  border-radius: 0.375rem;
  padding: 0.75rem;
  padding-left: 1.5rem;
  padding-right: 1.5rem;
}
.task:hover {
  box-shadow: none;
}
.labels {
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  width: 100%;
}
.card-content {
  display: grid;
  grid-column: 1;
  gap: 10px;
  justify-items: start;
  padding-bottom: 20px;
  padding-top: 20px;
}
.edit-button {
  background: #272343;
  color: white;
  border: none;
  width: 100%;
  padding: 10px;
  cursor: pointer;
  display: flex;
  justify-content: flex-end;
}
.edit-icon {
  margin-left: 10px;
}
.black-button {
  background-color: #272343;
  color: white;
  border: none;
  padding: 10px 20px 10px 20px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  cursor: pointer;
  border-radius: 5px;
  font-size: smaller;
  font-weight: bolder;
}
.black-button:hover {
  background-color: #e53170;
}
.drop-zone {
  background-color: #eee;
  margin-bottom: 10px;
  padding: 10px;
}
.attachments {
  display: flex;
  flex-direction: column;
}
.form-label-container {
  display: flex;
  flex-direction: column;
  border: 2px solid #272343;
  padding: 10px;
}
.form-labels {
  display: flex;
  flex-direction: row;
}
.add-label {
  border-radius: 0px;
  margin: 0px;
}
.label-preview-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  margin-top: 20px;
}
.label-preview {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  margin-top: 10px;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  font-weight: bolder;
  align-items: center;
}
.file {
  width: 100%;
  padding: 20px;
  background: #ffe9ed;
  margin-bottom: 20px;
  margin-right: 20px;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  font-weight: bolder;
  align-items: center;
}
.file-remove {
  background-color: #272343;
  color: white;
  border: none;
  padding: 10px 20px 10px 20px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  cursor: pointer;
  border-radius: 5px;
  font-size: smaller;
  font-weight: bolder;
}
.attachment-name {
  width: 100%;
  overflow: scroll;
}
.attachment-name::-webkit-scrollbar {
  width: 0;
  background: transparent;
}
.attachment-name::-webkit-scrollbar-thumb {
  background: #ff0000;
}
.attacments-preview {
  width: 100%;
}

.progress-bar {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  width: 100%;
  background: white;
  height: 20px;
  margin-top: 10px;
  padding: 12px;
  color: #272343;
  position: relative;
}
.progress-text {
  z-index: 10;
}
.progress-anchor {
  color: #e53170;
}

.progress-done {
  background: #e53170;
  position: absolute;
  top: 0;
  left: 0;
}
.card-item-title {
  font-weight: bold;
}
.custom-file-input {
  width: 100%;
  display: flex;
  padding: 10px;
  margin-bottom: 20px;
  margin-top: 20px;
  border: 2px solid #272343;
}
</style>
