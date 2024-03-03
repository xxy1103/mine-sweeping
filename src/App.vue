<template>
<h1 class="title">扫雷游戏</h1>
<hr/>
<p>
    <table align="center">
        <tr v-for="(rows,rowsIndex) in vmap" :key="rowsIndex" >
             <!-- 以从map中获取row以创建行，并以每个row的下标设为每个trb标签的标识-->
            <td v-for="(cols,colsIndex) in rows" 
             :key="colsIndex"  
             :class="hidemap[rowsIndex][colsIndex] === 0?isVisible(rowsIndex,colsIndex):mineJudge(rowsIndex,colsIndex)"
              @click.left="toggleVisibility(rowsIndex,colsIndex)"
              @contextmenu.prevent="mark(rowsIndex,colsIndex)"
              > {{cols }}</td>
        </tr>
    </table>
</p>
<p class="result">{{ result }}</p>
<button class="button" v-on:click="reset" align="center">重开一把</button>
</template>
<script setup lang="ts">
import{ref} from 'vue'
const result = ref('');
const row = 10;
const col = 10;
const mine = 10;
const hidemap = ref(Array.from(Array(row+2),() => Array(col+2).fill(0)))
const visibleMap = ref(Array.from(Array(row+2),() => Array(col+2).fill(false)))
const vmap = ref(Array.from(Array(row+2),() => Array(col+2).fill(0)))
initialise();
function mark(rowsIndex:any,colsIndex:any)
{
    if(visibleMap.value[rowsIndex][colsIndex] !== 'mark')
        visibleMap.value[rowsIndex][colsIndex] = 'mark';
    else
        visibleMap.value[rowsIndex][colsIndex] = false;
}
function mineJudge(i:any,j:any)
{
    if(visibleMap.value[i][j] === true)
        return 'mine';
    else if(visibleMap.value[i][j] === 'mark')
        return 'mark';
    else
        return 'shelter';
}
function isVisible(rowsIndex:any,colsIndex:any)
{
    if(rowsIndex === 0)
        return 'hiden';
    else if(rowsIndex === row+1)
        return 'hiden';
    else if(colsIndex === 0)
        return 'hiden';
    else if(colsIndex === col+1)
        return 'hiden';
    else if(visibleMap.value[rowsIndex][colsIndex] === true)
        return 'visible';
    else if(visibleMap.value[rowsIndex][colsIndex] === 'mark')
        return 'mark';
    else if(visibleMap.value[rowsIndex][colsIndex] === false)
        return 'shelter';
}
function open(rowsIndex:any,colsIndex:any)
{
    if(rowsIndex>=1 && rowsIndex <=row && colsIndex >=1 && colsIndex <=col )
    {
        if(vmap.value[rowsIndex+1][colsIndex] === ' ' && visibleMap.value[rowsIndex+1][colsIndex] != true)
        {
            visibleMap.value[rowsIndex+1][colsIndex] = true;
            open(rowsIndex+1,colsIndex);
        }
        if(vmap.value[rowsIndex][colsIndex+1] === ' ' && visibleMap.value[rowsIndex][colsIndex+1] != true)
        {
            visibleMap.value[rowsIndex][colsIndex+1] = true;
            open(rowsIndex,colsIndex+1);
        }
        if(vmap.value[rowsIndex][colsIndex-1] === ' ' && visibleMap.value[rowsIndex][colsIndex-1] != true)
        {
            visibleMap.value[rowsIndex][colsIndex-1] = true;
            open(rowsIndex,colsIndex-1);
        }
        if(vmap.value[rowsIndex-1][colsIndex] === ' ' && visibleMap.value[rowsIndex-1][colsIndex] != true)
        {
            visibleMap.value[rowsIndex-1][colsIndex] = true;
            open(rowsIndex-1,colsIndex);
        }
    }
    return 1;
}
function toggleVisibility(rowsIndex:any,colsIndex:any)
{
    visibleMap.value[rowsIndex][colsIndex] = true;
    if(vmap.value[rowsIndex][colsIndex] === ' ')
        open(rowsIndex,colsIndex);
    else if(hidemap.value[rowsIndex][colsIndex] === 1)
    {
        lose();
    }
    judgeWin();
}
function judgeWin()
{
    let mineCount = 0;
    for(let i = 1;i<=row;i++)
    {
        for(let j = 1;j<=col;j++)
        {
            if(visibleMap.value[i][j] === false)
            {
                    mineCount++;
            }
        }
    }
    if(mineCount === mine)
    {
        win();
    }
}

function initialise()
{
    let lmine = 0
    let now = new Date();
    result.value = '';
    for(let i = 1;i<=row;i++)
    {
        for(let j = 1;j<=col;j++)
        {
            hidemap.value[i][j] = 0;
        }
    }
    while(lmine < mine)
    {
    let randomRow = Math.floor(Math.random()*row+1)
    let randomCol = Math.floor(Math.random()*col+1)
    if(hidemap.value[randomRow][randomCol] === 0)
    {
        hidemap.value[randomRow][randomCol] = 1
        lmine++
    }
}
    for(let i = 1;i<=row;i++)
    {
       for(let j = 1;j<=col;j++)
        {
            vmap.value[i][j] = hidemap.value[i+1][j+1] + hidemap.value[i+1][j] + hidemap.value[i+1][j-1] + hidemap.value[i][j+1] + hidemap.value[i][j-1] + hidemap.value[i-1][j-1] + hidemap.value[i-1][j] + hidemap.value[i-1][j+1];
            if(hidemap.value[i][j])
            {
                vmap.value[i][j] = "雷";
            }
            if(vmap.value[i][j] === 0)
            {
                vmap.value[i][j] = ' ';
            }
        }
    }
}
function lose()
{
    for(let i = 1;i<=row;i++)
    {
        for(let j = 1;j<=col;j++)
        {
            visibleMap.value[i][j] = true;
        }
    }
    result.value = "You lose!!!"
}
function win()
{
    for(let i = 1;i<=row;i++)
    {
        for(let j = 1;j<=col;j++)
        {
            visibleMap.value[i][j] = true;
        }
    }
    result.value = 'You win!!!!!!!!!!'
}
function reset()
{
    initialise();
    for(let i = 1;i<=row;i++)
    {
        for(let j = 1;j<=col;j++)
        {
            visibleMap.value[i][j] = false;
        }
    }
}
</script>