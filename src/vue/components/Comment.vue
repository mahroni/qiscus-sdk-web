<template>
  <div class="qcw-comment-container" :id="comment.id">
    <div class="qcw-comment-date" v-if="showDate">
      - {{ dateToday }} - 
    </div>
    <div class="qcw-comment" :class="{
      'comment--me': comment.username_real == myemail,
      'comment--parent': isParent,
      'comment--mid': isMid,
      'comment--last': isLast
    }">
      <!-- <avatar :src="comment.avatar"></avatar> -->
      <div class="qcw-comment__message">
        <div class="qcw-comment__info" v-if="isParent">
          <span class="qcw-comment__username">{{comment.username_as}}</span>
          <span class="qcw-comment__time" v-if="isParent">{{comment.time}}</span>
        </div>
        <!-- CommentType: "TEXT" -->
        <div v-if="comment.type == 'text'">
          <image-loader v-if="comment.isAttachment()"
            :comment="comment"
            :on-click-image="onClickImage"
            :callback="onupdate">
          </image-loader>
          <div v-html="message" v-if="!comment.isAttachment()" class="qcw-comment__content"></div>
          <span class="qcw-comment__time qcw-comment__time--children" i
            v-if="!isParent"
            :class="{'qcw-comment__time--attachment': comment.isAttachment}">{{comment.time}}</span>
          <div v-if="isMe">
            <i class="qcw-comment__state fa fa-clock-o" v-if="comment.isPending"></i>
            <i class="qcw-comment__state fa fa-check" v-if="comment.isSent && !comment.isDelivered"></i>
            <div class="qcw-comment__state qcw-comment__state--delivered" v-if="comment.isDelivered && !comment.isRead">
              <i class="fa fa-check"></i><i class="fa fa-check"></i>
            </div>
            <div class="qcw-comment__state qcw-comment__state--read" v-if="comment.isRead">
              <i class="fa fa-check"></i><i class="fa fa-check"></i>
            </div>
          </div> <!-- end of comment icons -->
        </div>
        <!-- CommentType: "ACCOUNT_LINKING" -->
        <div v-if="comment.type == 'account_linking'">
          <div v-html="message" class="qcw-comment__content"></div>
          <div class="action_buttons">
            <button @click="openAccountBox">{{ comment.payload.params.button_text }} &rang;</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import EmbedJS from 'embed-js';
import marked from 'marked';
import ImageLoader from './ImageLoader.vue';
// import highlight from 'highlight.js';
// import Avatar from './Avatar';

export default {
  // components: { Avatar },
  props: ['comment','onupdate', 'onClickImage', 'commentBefore', 'commentAfter'],
  components: { ImageLoader },
  updated(){
    // this.onupdate();
  },
  computed: {
    myemail() { return qiscus.email },
    isParent() { return this.commentBefore == null || this.commentBefore.username_real != this.comment.username_real; },
    isMid() { return this.commentAfter != null && !this.isParent && this.commentAfter.username_real == this.comment.username_real; },
    isLast() { return this.commentAfter == null || this.commentAfter.username_real != this.comment.username_real; },
    isMe () { return this.comment.username_real == this.myemail },
    showDate() { return this.commentBefore === null || (this.commentBefore.date != this.comment.date) },
    renderedComment() { return (typeof emojione != "undefined") ? emojione.toShort(this.comment.message) : this.comment.message }
  },
  created() {
    if(!this.comment.isAttachment()) {
      this.message = this.comment.message
      this.x.text((data) => {
        this.message = data;
        this.onupdate();
      });
    }
  },
  methods: {
    openAccountBox() {
      window.open(this.comment.payload.url, 'AccountLinkingPopup', 'width=500,height=400,location=no,menubar=no,resizable=1,status=no,toolbar=no')
    }
  },
  data () {
    return {
      message: '',
      dateToday: new Date(this.comment.date).toLocaleString('en-US', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' }),
      me: qiscus.email,
      x: new EmbedJS({
        input: (typeof emojione != 'undefined') ? emojione.toShort(this.comment.message) : this.comment.message,
        openGraphEndpoint: `${qiscus.baseURL}/api/v2/mobile/get_url_metadata?url=$\{url\}`,
        googleAuthKey: 'AIzaSyAO1Oui55SvTwdk4XCMzmAgr145urfQ9No',
        inlineEmbed: 'all',
        onOpenGraphFetch: function(data) {
          const title = data.results.metadata.title
          const isTitleExists = title && title.length > 0
          const isSuccess = data.results.found && isTitleExists
          const objectGraph = {
            title: data.results.metadata.title,
            description: data.results.metadata.description,
            image: data.results.metadata.image,
            url: data.results.url,
            type: 'site',
            success: isSuccess
          }
          setTimeout( () => this.onupdate(), 0 )
          return objectGraph
        }.bind(this),
        marked: true,
        emoji: true,
        plugins: {
          marked: marked,
        }
      })
    }
  }
}
</script>