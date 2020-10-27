<template>
    <div>
        <!-- 1 -->
    </div>
</template>
<script>
//递归函数 实现100累加
function countNum(num) {
    if (num <= 1) {
        return true;
    }
    return countNum - 1 + num;
}
console.log(countNum(100));
</script>