<!-- 显示通知具体内容 -->

<script>
  import { currentUserEmail, currentnoticeid, isJoinedTodo } from "../store.js";
  import PocketBase from "pocketbase";
  import { PocketBase_URL } from "../utils/api/index";
  import { onMount } from "svelte";
  import { writable } from "svelte/store";

  const pb = new PocketBase(PocketBase_URL);
  let records = [];
  let ifJoined = writable(false);
  async function noticedisplay() {
    if ($isJoinedTodo == "find") {
      ifJoined.set(true);
    }
    //alert("第一个函数被调用")
    try {
      const createid = $currentnoticeid;
      const response = await pb.collection("notices").getFullList({
        sort: "-created",
        filter: `id="${createid}"`,
      });
      records = response;
    } catch (error) {
      alert("fail to find");
    }
  }
  async function copy(check) {
    ifJoined.set(true);
    try {
      const userEmail = $currentUserEmail;
      const newRecord = {
        tittle: check.tittle,
        body: check.body,
        tag: check.tag,
        year: check.year,
        month: check.month,
        day: check.day,
        useremail: userEmail,
        noticeid: check.id,
        username: check.username,
      };
      await pb.collection("todolist").create(newRecord);
      alert("添加成功！");
    } catch (error) {
      alert("添加失败。");
    }
  }
  onMount(() => {
    noticedisplay();
  });
</script>

{#each records as record}
  <div class="record">
    <div class="tittle">{record.tittle}</div>
    <div class="meta">
      {record.year}.{record.month}.{record.day}/#{record.tag}/from:{record.username}
    </div>
    <div>{record.body}</div>
    {#if $ifJoined == false}
      <button class="but" on:click={() => copy(record)}>+ todolist</button>
    {:else}
      <button class="but">已加入todolist</button>
    {/if}
  </div>
{/each}

<style>
  .record {
    position: fixed;
    top: 0%;
    width: auto;
    height: 90%;
    border: 1px solid #ccc;
    padding: 15px;
    margin: 4%;
    background-color: #f9f9f9;
  }

  .tittle {
    font-size: 40px; /* 标题字体大小 */
    font-weight: bold; /* 加粗显示 */
    margin-bottom: 10px; /* 标题与其他内容之间增加空间 */
    color: #333; /* 深色字体，增加对比，易于阅读 */
  }

  .meta {
    font-size: 14px; /* 元数据使用更小的字体 */
    color: #666; /* 浅色字体，区分正文 */
    margin-bottom: 10px; /* 与正文内容分隔 */
  }

  .but {
    color: black; /* 设置字体颜色为黑色 */
    border: 2px solid black; /* 2px宽度的实线边框，颜色为黑色 */
  }
</style>