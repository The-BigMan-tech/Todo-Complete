<script lang="ts">
    import { onMount } from "svelte";

    interface TaskData {
        name:string
    }
    let task:string[] = $state([])
    let taskData:TaskData[] = $state([])
    let checked:string[] = $state([])
    let cross:string[] = $state([])
    let edit:boolean[] = $state([])
    let edit_or_done:string[] = $state([])

    function typingTask(event:Event,index:number):void {
        const target = event.target as HTMLInputElement
        task[index] = target.value
    }
    function toggleCrossTask(index:number):void {
        if (checked[index]) {
            cross[index] = 'line-through text-[#9e9c9c]'
            return
        }
        cross[index] = ''
    }
    function toggleCheckedTask(index:number):void {
        if (!checked[index]){
            checked[index] = 'bg-[#031E6F]'
            toggleCrossTask(index)
            return
        }
        checked[index] = ''
        toggleCrossTask(index)
    }
    async function loadChanges() {
        const response = await fetch('http://localhost:4100/getTask',{method:'GET'})
        if (!response.ok) throw new Error('Got an error on response')
        taskData = await response.json()
    }
    onMount(()=>{
        loadChanges()
    })
    async function editTask(index:number) {
        console.log(`RECEIVED THE NEW TASK: ${task[1]} AT INDEX: ${index}`);
        await fetch(`http://localhost:4100/editTask/${encodeURIComponent(JSON.stringify({name:task[1],index:index}))}`,
            {method:'PUT',}
        )
        loadChanges()
    }
    function toggleEditTask(index:number,defaultText:string):void {
        if (edit[index]) {
            edit[index] = false
            edit_or_done[index] = '/assets/images/pencil-solid(1).svg'
            editTask(index)
            return
        }
        edit[index] = true
        edit_or_done[index] = '/assets/images/square-check-solid.svg'
        task[1] = defaultText as string
    }
    async function addTask() {
        if (!task[0].length) return;
        await fetch('http://localhost:4100/addTask',
        {
            method:'POST',
            headers:{'Content-Type':'application/json'},
            body:JSON.stringify({name:task[0]})
        })
        loadChanges()
    }
    async function deleteTask(removeTask:TaskData,index:number) {
        await fetch(
            `http://localhost:4100/deleteTask/${encodeURIComponent(JSON.stringify(removeTask))}`,
            {method:'DELETE'}
        )
        loadChanges()
        if (checked[index]) toggleCheckedTask(index)
    }
</script>
<div class="flex justify-center relative top-24">
    <div class="flex flex-col items-center">
        <h1 class="text-[#00e5ff] text-7xl font-[800]">TODO APP</h1>
        <div class="flex relative">
            <div class="mt-10">
                <form action="">
                    <input onchange={(event)=>typingTask(event,0)} value='' class="py-3 px-6 border-[#B7D8E8] w-[35rem] border rounded-3xl text-xl font-[Consolas] outline-none bg-transparent text-[#B7D8E8]" type="text" placeholder="You can write anything here">
                    <button onclick={addTask}>
                        <div class="bg-[#00e5ff] py-3 px-8 font-[600] text-lg rounded-3xl absolute right-0 top-[2.55rem]">
                            <h1>ADD</h1>
                        </div>
                    </button>
                </form>
                <div class="flex flex-col gap-5 relative mt-8">
                    {#each taskData as addedTask,index}
                        <div class="flex bg-[#51587e] relative rounded-2xl py-4 px-6 items-center gap-5 text-white">
                            <button onclick={()=>toggleCheckedTask(index)} class="relative">
                                <div class={`border-2 border-[#00e5ff] h-6 w-6 rounded-sm text-transparent ${checked[index]}`}>0</div>
                                {#if (checked[index])}
                                    <img class="absolute w-3 h-3 left-[6px] top-[5px]" src="/assets/images/check-solid(1).svg" alt="">
                                {/if}
                            </button>
                            {#if (edit[index])}
                                <input onchange={(event)=>typingTask(event,1)} value={addedTask.name} class={`text-xl text-black outline-none w-96 ${cross[index]}`}/>
                            {:else}
                                <h1 class={`text-xl outline-none bg-transparent ${cross[index]}`}>{addedTask.name}</h1>
                            {/if}
                            <div class="flex absolute right-4 gap-5">
                                <button onclick={()=>toggleEditTask(index,addedTask.name as string)}>
                                    {#if (edit_or_done[index])}
                                        <img class="w-4" src={edit_or_done[index]} alt="">
                                    {:else}
                                        <img class="w-4" src="/assets/images/pencil-solid(1).svg" alt="">
                                    {/if}
                                </button>
                                <button onclick={()=>deleteTask(addedTask,index)}>
                                    <img class="w-4" src="/assets/images/trash-can-solid.svg" alt="">
                                </button> 
                            </div>
                        </div> 
                    {/each}
                </div>
            </div>
        </div>
    </div>
</div>
