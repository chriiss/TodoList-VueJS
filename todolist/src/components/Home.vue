<template>
  <div class="home" v-bind:class="{ 'theme-dark': nightMode }">
    <header v-bind:class="{ 'theme-dark': nightMode }">
      <form v-on:submit.prevent="addTask">
        <input
          type="text"
          v-bind:class="{
            'theme-dark': nightMode,
            'is-invalid': $v.task.$error,
            'is-valid': !$v.task.$invalid,
          }"
          placeholder="Saisir votre tâche"
          v-model="task"
          @blur="$v.task.$touch()"
        />
        <button type="submit" v-bind:class="{ invalid: $v.$invalid }">
          <i class="fa fa-plus" aria-hidden="true"></i>
        </button>
      </form>
    </header>
  
    <section>
      <ul>
        <li
          v-bind:class="{ 'theme-dark': nightMode, completed: task.done }"
          v-for="task in tasks.slice().reverse()"
          track-by="$index"
          :key="task.id"
        >
          <div class="task">{{ task.description }}</div>
          <div class="flex_icon">
            <div>
              <i
                class="fa fa-check-square"
                v-bind:class="{ check: task.done }"
                v-on:click="task.done = !task.done"
                aria-hidden="true"
              ></i>
            </div>

            <div>
              <i class="fa fa-trash" aria-hidden="true" v-on:click="removeTask(task)"></i>
            </div>

            <div v-if="!task.isActiveEditMode">
              <i class="fa fa-pencil-square" v-on:click="switchMode(task)" aria-hidden="true"></i>
            </div>

            <div v-else>
              <input
                type="text"
                v-bind:class="{ 'theme-dark': nightMode }"
                v-model="task.description"
                v-on:keyup.enter="switchMode(task)"
              />
              <i class="fa fa-pencil-square" aria-hidden="true" v-on:click="switchMode(task)"></i>
            </div>
          </div>
          <div class="date-bloc">{{date}} {{ord}} {{day}} {{year}}</div>
        </li>
      </ul>
    </section>

    <footer>
      <button type="button" v-on:click="deleteAll">
        <span v-for="data in dataWithJson" :key="data.id">{{data.deletetext }}</span>
      </button>
      <div>
        <div v-bind:class="{ isChecked }" v-if="isChecked">
          <p v-for="data in dataWithJson" :key="data.id">{{ data.nighttext }}</p>
        </div>
        <div v-else>
          <p v-for="data in dataWithJson" :key="data.id">{{ data.daytext }}</p>
        </div>
        <label class="switch" for="theme-toggle">
          <input type="checkbox" @change="changeText" id="theme-toggle" v-model="nightMode" />
          <span class="slider"></span>
        </label>
      </div>
    </footer>
  </div>
</template>

<script>
import jsonFile from "../datas/data.json";
import { required, minLength } from "vuelidate/lib/validators";
export default {
  name: "home",
  data() {
    return {
      dataWithJson: jsonFile,
      task: null,
      tasks: [],
      nightMode: null,
      isChecked: false,
      day: this.taskDay(),
      date: new Date().getDate(),
      ord: this.nth(new Date().getDate()),
      year: new Date().getFullYear()
    };
  },
  validations: {
    task: {
      required,
      minLength: minLength(1)
    }
  },
  mounted() {
    if (
      localStorage.getItem("task") &&
      localStorage.getItem("nightMode") &&
      localStorage.getItem("isChecked")
    ) {
      this.tasks = JSON.parse(localStorage.getItem("task"));
      this.nightMode = JSON.parse(localStorage.getItem("nightMode"));
      this.isChecked = JSON.parse(localStorage.getItem("isChecked"));
    }
  },
  watch: {
    tasks: {
      handler() {
        localStorage.setItem("task", JSON.stringify(this.tasks));
      },
      deep: true
    },

    nightMode() {
      localStorage.setItem("nightMode", JSON.stringify(this.nightMode));
    },

    isChecked() {
      localStorage.setItem("isChecked", JSON.stringify(this.isChecked));
    }
  },
  methods: {
    addTask() {
      this.$v.$touch();
      if (this.$v.$invalid) {
        alert("The field is empty");
      } else {
        this.tasks.push({
          id: Date.now(),
          description: this.task,
          done: false,
          isActiveEditMode: false
        });
        this.task = null;
      }
    },
    taskDay() {
      var d = new Date();
      var days = [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday"
      ];
      return days[d.getDay()];
    },
    nth(d) {
      if (d > 3 && d < 21) {
        return "th";
      }
      switch (d % 10) {
        case 1:
          return "st";
        case 2:
          return "nd";
        case 3:
          return "rd";
        default:
          return "th";
      }
    },
    switchMode(task) {
      task.isActiveEditMode = !task.isActiveEditMode;
    },

    removeTask(task) {
      this.tasks = this.tasks.filter(function(task_bis) {
        return task_bis.id !== task.id;
      });
    },

    deleteAll() {
      alert("Do you want to clean the tasklist?");
      for (let i; this.tasks.length > 0; i--) {
        this.tasks.pop(i);
      }
    },

    changeText() {
      this.isChecked = !this.isChecked;
    }
  }
};
</script>

<style lang="scss">
@import "../styles/Variables.scss";
.home {
  background-color: $day_mode;
  min-height: 100vh;
  input[type="text"] {
    height: $size_input;
    width: 80%;
    color: $color_input;
    border-top: none;
    border-left: none;
    border-right: none;
    outline: none;
    border-bottom: 1px dashed $color_input;

    &:hover {
      border-bottom: 1px dashed $color_check;
    }

    &.theme-dark {
      background: $night_mode;
      color: $day_mode;
      border-bottom: 1px dashed $day_mode;
    }
  }

  header {
    form {
      margin-left: 10px;

      ::placeholder {
        color: $color_input;
      }

      button[type="submit"] {
        border: none;
        height: $size_input;
        width: $size_input;
        border-radius: $size_input;
        background-color: $color_check;
        color: $color_font;
        font-size: 20px;
        padding-top: 5px;
        outline: none;
        margin-top: 10px;

        &:hover {
          background-color: $color_font;
          color: $color_check;
          border: 1px dashed $color_check;
        }
      }

      .invalid {
        opacity: 0.5;
      }
    }

    .invalid_feedback {
      margin: 15px 0px 0px 15px;
      color: $color_trash;
      font-weight: bold;
    }
  }
  section {
    padding-bottom: 95px;
    ul {
      li {
        list-style: none;
        cursor: pointer;
        margin-top: 47.5px;
        text-align: center;
        border: 1px dashed $color_input;
        padding: 23.75px;
        width: 80%;
        margin-right: auto;
        margin-left: auto;
        background-color: $color_font;

        &.theme-dark {
          background: $night_mode;
          border: 1px dashed $day_mode;
          color: $day_mode;
        }

        .flex_icon {
          display: flex;
          justify-content: space-around;

          div {
            margin-top: 23.75px;
          }
        }
        .check {
          color: $color_check;
        }

        .fa-trash {
          color: $color_trash;
        }

        input[type="text"] {
          background-color: $color_font;
          margin-top: 20px;
          &.theme-dark {
            background: $night_mode;
            color: $day_mode;
          }
        }

        .fa-pencil-square:first-child {
          color: #8000ff;
        }

        .fa-pencil-square:nth-child(2) {
          margin-left: 20px;
          color: $color_check;
        }

        .date-bloc {
          font-size: 12px;
          float: left;
          margin-top: 5px;
        }
      }
      .completed {
        text-decoration: line-through;
      }
    }
  }
  footer {
    position: fixed;
    left: 0;
    bottom: 0;
    width: 100%;
    display: flex;
    div:nth-child(2) {
      display: block;
      margin: auto;
    }
    .isChecked {
      color: $color_font;
    }
    button[type="button"] {
      background-color: $color_trash;
      color: $color_font;
      border: none;
      height: 50px;
      font-size: 20px;
      width: 50%;

      &:hover {
        background-color: $color_font;
        color: $color_trash;
        border: 1px dashed $color_trash;
      }
    }

    .switch {
      position: relative;
      display: inline-block;
      width: 60px;
      height: 34px;
    }

    .switch input {
      opacity: 0;
      width: 0;
      height: 0;
    }

    .slider {
      position: absolute;
      cursor: pointer;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background-color: #ccc;
      -webkit-transition: $switchTime_transition;
      transition: $switchTime_transition;
    }

    .slider:before {
      position: absolute;
      content: "";
      height: 26px;
      width: 26px;
      left: 4px;
      bottom: 4px;
      background-color: $day_mode;
      -webkit-transition: $switchTime_transition;
      transition: $switchTime_transition;
    }

    input:checked + .slider {
      background-color: $color_check;
    }

    input:focus + .slider {
      box-shadow: 0 0 1px $color_check;
    }

    input:checked + .slider:before {
      -webkit-transform: translateX($translateX_size);
      -ms-transform: translateX($translateX_size);
      transform: translateX($translateX_size);
    }
    .slider.round {
      border-radius: 34px;
    }

    .slider.round:before {
      border-radius: 50%;
    }
  }
  &.theme-dark {
    background: $night_mode;
  }
}
</style>
