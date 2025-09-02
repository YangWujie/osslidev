---
theme: seriph
class: text-center
background: https://cover.sli.dev

title: 操作系统引论
---

# tst 

a 

---

```ts {2|3|7|12}{maxHeight:'100px'}
function add(
  a: Ref<number> | number,
  b: Ref<number> | number
) {
  return computed(() => unref(a) + unref(b))
}
/// ...as many lines as you want
const c = add(1, 2)
```

---

<style global>
.slidev-layout {
  font-size: 28px;
}

.slidev-layout h1 {
  font-size: 64px; /* Set font size for h1 (main title) */
}

.slidev-layout h2 {
  font-size: 52px; /* Set font size for h2 (subtitles) */
}

.slidev-layout h3 {
  font-size: 40px; /* Set font size for h3, if needed */
}

/* 确保标题和内容的间距 */
.slidev-layout h1, 
.slidev-layout h2, 
.slidev-layout h3 {
  margin-top: 2.5rem;
  margin-bottom: 3.5rem;
}

.slidev-layout p {
  line-height: 1.5;
}

</style>