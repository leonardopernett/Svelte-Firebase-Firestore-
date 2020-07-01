<script>
	import {firestore} from 'firebase';
	import {onMount} from 'svelte'
	import toastr from 'toastr'

	 let tasks =[];
	 let task = {name:"",description:""}
	 let id =""
   let editStatus = false;

const addTask = async (task)=>{
  await firestore().collection('tasks').doc().set({...task, createAt:Date.now()})
  console.log("task saved")
}

const handlerSubmit = ()=>{
    addTask(task)
    task.name=""
		task.description=""
		toastr.success('task added successfully',{
			"progressBar": true,
			"timeOut": "5000",
			"closeButton": true,
		});
}
	
firestore().collection('tasks').onSnapshot(queryShapshot=>{
		let docs=[]
		queryShapshot.forEach(doc=>{
				docs.push({...doc.data(), id:doc.id})
		})
		tasks = docs;
		
})

	const deleteTask = async (id)=>{
		if(confirm('are sure sure that delete')){
			await firestore().collection('tasks').doc(id).delete();
			toastr.error('task delete successfully',{
			"progressBar": true,
			"timeOut": "5000",
			"closeButton": true,
		});
		}
	}

const editTask = async(currentTask)=>{
  	editStatus= true
		task.name=currentTask.name
		task.description= currentTask.description
		id= currentTask.id
}

const updateTask = async (e)=>{
		await firestore().collection('tasks').doc(id).update(task)
			toastr.success('task updated successfully',{
			"progressBar": true,
			"timeOut": "5000",
			"closeButton": true,
		});
		editStatus= false;
    e.target.reset()
}

const onCancel = (e)=> {
	editStatus= false;
	task = {name:"",description:""}
}
	
</script>


<div class="container">
  <div class="row">
	  <div class="col-md-12 mx-auto">

		  <div class="card">
			   <div class="card-body">
				     <form on:submit|preventDefault={editStatus ? updateTask : handlerSubmit }>
								<div class="form-group">
								<input bind:value={task.name} type="text" class="form-control" placeholder="write task">
								</div>
								<div class="form-group">
								<textarea bind:value={task.description} placeholder="descrition task" class="form-control" rows="3"></textarea>
								</div>
							  {#if editStatus===true}
                  <button class="btn btn-dark">update</button>
									<button class="btn btn-dark" on:click={onCancel}>cancel</button>
								{:else}
								  <button class="btn btn-primary">save</button>
								{/if}
							</form>
				 </div>
			</div>
		</div>

		<div class="col-md-12">	
			 {#each tasks as task (task.id) }
			    <div class="card">
				    <div class="card-body">
					  	<div class="d-flex justify-content-between align-items-center">
							<h3>{task.name}</h3>
							 <i on:click={editTask(task)} class="text-dark material-icons">create</i>
							</div>
								<p>{task.description}</p>
								<button class="btn btn-danger btn-block btn-sm" on:click={deleteTask(task.id)}>
								  <i class="material-icons">delete</i>
								</button>
								
						</div>
				  </div>
			 {/each}
		</div>
	</div>
</div>

