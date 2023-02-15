<template>
  <div class="discussion">
    <div v-for="comment in comments" :key="comment.id" class="comment">
      <div class="comment-header">
        <img
          v-if="comment.user.avatar"
          :src="comment.user.avatar"
          :alt="comment.user.name"
          class="avatar"
        />
        <!-- PlaceHolder image in case Avatar doesn't exist -->
        <img
          v-else
          src="https://upload.wikimedia.org/wikipedia/commons/7/7c/Profile_avatar_placeholder_large.png"
          class="avatar"
        />
        <span class="name">{{ comment.user.name }}</span>
        <span class="date">{{ formatDate(comment.date) }}</span>
      </div>
      <div class="comment-text">{{ comment.text }}</div>
      <div class="comment-actions">
        <!-- Like Button -->
        <button class="like-button" @click="toggleLike(comment)">
          <i :class="['like-icon', { liked: comment.iLikedIt }]"></i>
          <span class="likes">{{ comment.likes }}</span>
        </button>
        <!--Reply Button  -->
        <button class="reply-button" @click="showReply(comment)">
          <i class="reply-icon"></i>
          <span class="reply-text">Reply</span>
        </button>
      </div>
      <div v-if="comment.replies">
        <div v-if="comment.replies.length" class="replies">
          <Discussion
            :comments="comment.replies"
            :show-reply-form="showReplyForm === comment.id"
            @reply="addReply"
          ></Discussion>
        </div>
      </div>

      <div v-if="showReplyForm === comment.id" class="reply-form">
        <img src="../assets/userAvatar.jpeg" class="avatar" />
        <input
          type="text"
          v-model="newReplyText"
          class="reply-textarea"
          placeholder="Write a reply..."
          v-on:keyup.enter="submitReply(comment.id)"
          rows="2"
        />
        <button class="reply-submit-button" @click="submitReply(comment.id)">
          Submit
        </button>
      </div>
    </div>
  </div>
</template>
  
  <script lang="ts">
import { defineComponent } from "vue";
import type { IDiscussion, IComment } from "./src/types/IDiscussion";

export default defineComponent({
  name: "Discussion",
  // Props
  props: {
    comments: {
      type: Array as () => IComment[],
      required: true,
    },
    showReplyFormProp: {
      type: Boolean,
      default: false,
    },
  },
  // Emitted events
  emits: ["reply"],
  // Data
  data() {
    return {
      newReplyText: "",
      showReplyForm: this.showReplyFormProp,
    };
  },
  // Watchers
  watch: {
    showReplyFormProp(newValue) {
      this.showReplyForm = newValue;
    },
  },
  // Methods
  methods: {

    // Formats the date to a readable format
    formatDate(date: number) {
      const options = { year: "numeric", month: "long", day: "numeric" };
      return new Date(date).toLocaleDateString(undefined, options);
    },

    toggleReplyForm() {
      this.showReplyForm = !this.showReplyForm;
    },
    toggleLike(comment: IComment) {
      if (comment.iLikedIt) {
        comment.iLikedIt = false;
        comment.likes--;
      } else {
        comment.iLikedIt = true;
        comment.likes++;
      }
    },
    showReply(comment: IComment) {
      this.showReplyForm = comment.id;
    },
    submitReply(commentId: number) {
      if (!this.newReplyText) {
        return;
      }
      const newReply: IComment = {
        id: Math.floor(Math.random() * 10000) + 1,
        date: Date.now(),
        user: {
          name: "User",
          avatar: "https://www.github.com/yyx990803.png",
        },
        text: this.newReplyText,
        likes: 0,
        iLikedIt: false,
        replies: [],
      };
      console.log("newReply", newReply);
      this.$emit("reply", commentId, newReply);
      this.newReplyText = "";
      this.showReplyForm = false;
    },
    addReply(commentId: number, newReply: IComment) {
      const comment = this.comments.find((c) => c.id === commentId);
      if (comment) {
        comment.replies.push(newReply);
      }
      this.$emit("reply", commentId, newReply);
    },
  },
});
</script>
  
  <style lang="scss">
.discussion {
  margin: 1rem 0 0 2rem;
  padding: 1rem;
  padding-left: 2rem;
  border-left: 1px solid #ccc;

  .comment {
    margin: 0 0 2rem;
    // border-bottom: 1px solid #ccc;

    .comment-header {
      display: flex;
      align-items: center;
      margin: 0 0 1rem;

      .avatar {
        width: 2rem;
        height: 2rem;
        margin: 0 1rem 0 0;
        border-radius: 50%;
      }

      .name {
        font-weight: bold;
      }

      .date {
        margin: 0 0 0 1rem;
        color: #666;
      }
    }

    .comment-text {
      margin: 0 0 1rem;
    }

    .comment-actions {
      display: flex;
      align-items: center;

      .like-button {
        display: flex;
        align-items: center;
        margin: 0 1rem 0 0;
        padding: 0;
        border: none;
        background: none;
        cursor: pointer;

        .like-icon {
          transition: all 0.2s ease-in-out;
          width: 1.5rem;
          height: 1.5rem;
          margin: 0 0.5rem 0 0;
          background: url("../assets/like.svg") no-repeat center center;
          background-size: contain;
          cursor: pointer;
          &:hover {
            transform: scale(1.2);
          }
          &.liked {
            background: url("../assets/liked.svg") no-repeat center center;
            background-size: contain;
          }
        }

        .likes {
          color: #666;
        }
      }

      .reply-button {
        display: flex;
        align-items: center;
        margin: 0 1rem 0 0;
        padding: 0;
        border: none;
        background: none;
        cursor: pointer;
        .reply-text {
          color: #0d6efd;
          &:hover {
            color: blue;
          }
        }
      }
    }
    .reply-form {
      display: flex;
      margin-top: 1rem;
      .reply-textarea {
        background-color: rgb(250, 251, 252);
        border-color: rgb(223, 225, 230);
        color: rgb(9, 30, 66);
        cursor: text;
        margin-right: 4px;
        border-radius: 3px;
        border-width: 2px;
        border-style: solid;
        box-sizing: border-box;
        font-size: 14px;
        transition: background-color 0.2s ease-in-out 0s,
          border-color 0.2s ease-in-out 0s;
        line-height: 1.42857;
        padding: 8px 6px;
        height: 2.5rem;
        &:hover {
          background-color: rgb(235, 236, 240);
        }
        &:focus {
          background-color: rgb(255, 255, 255);
          border-color: rgb(76, 154, 255);
        }
      }
      .reply-submit-button {
        cursor: pointer;
        outline: 0;
        display: inline-block;
        font-weight: 400;
        line-height: 1.5;
        text-align: center;
        background-color: transparent;
        border: 1px solid transparent;
        padding: 6px 12px;
        font-size: 1rem;
        border-radius: 0.25rem;
        transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out,
          border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
        color: #0d6efd;
        border-color: #0d6efd;
        &:hover {
          color: #fff;
          background-color: #0d6efd;
          border-color: #0d6efd;
        }
      }
      .avatar {
        width: 2rem;
        height: 2rem;
        margin: 0 1rem 0 0;
        border-radius: 50%;
      }
    }
  }
}
</style>