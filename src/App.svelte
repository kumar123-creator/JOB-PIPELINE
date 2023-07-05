<script>
	import { onMount } from "svelte";
	import { createEventDispatcher } from "svelte";
	import "bootstrap/dist/css/bootstrap.min.css";
  
	let jsonData = [];
	let tableVisible = false;
	let selectedJob = null;
	let idList = [];
	let selectedCandidate = null;
  
	const dispatch = createEventDispatcher();
  
	async function fetchData() {
	  const response = await fetch("https://api.recruitly.io/api/job?apiKey=TEST64518616D4CF145D4E20BD47169EA7229BA3");
	  const responseData = await response.json();
	  jsonData = responseData.data;
	}
  
	async function fetchIdDetails(jobId) {
	  try {
		const response = await fetch(
		  `https://api.recruitly.io/api/job/pipeline/byjob?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77&jobApplication=false&jobId=${jobId}&rejected=false`
		);
  
		if (response.ok) {
		  const { data } = await response.json();
		  console.log(data);
		  if (Array.isArray(data)) {
			idList = data.map((item) => ({
			  id: item.id,
			  jobRef: item.jobRef,
			  jobTitle: item.jobTitle,
			  candidateLabel: item.candidateLabel,
			}));
		  } else {
			console.error("Error: Response data is not an array", data);
		  }
		} else {
		  console.error("Error:", response.status);
		}
	  } catch (error) {
		console.error("Error:", error);
	  }
	}
  
	async function fetchCandidateDetails(candidateId) {
	  try {
		const response = await fetch(
		  `https://api.recruitly.io/api/candidate/${candidateId}?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`
		);
  
		if (response.ok) {
		  const { data } = await response.json();
		  // Handle candidate details data
		  console.log(data);
		} else {
		  console.error("Error:", response.status);
		}
	  } catch (error) {
		console.error("Error:", error);
	  }
	}
  
	function handleTitleClick(job) {
	  selectedJob = job;
	  fetchIdDetails(job.id);
	 
	}
  
	function openCandidateDetails(candidate) {
	  selectedCandidate = candidate;
	  fetchCandidateDetails(candidate.id); // Fetch candidate details
	}
  
	onMount(async () => {
	  await fetchData();
	  tableVisible = true;
	});
  </script>
  
  <style>
	.popup {
	  position: fixed;
	  top: 0;
	  left: 0;
	  width: 100%;
	  height: 100%;
	  background-color: rgba(0, 0, 0, 0.5);
	  display: flex;
	  justify-content: center;
	  align-items: center;
	}
  
	.popup-content {
    background-color: rgb(242, 243, 244);
    padding: 20px;
    border-radius: 5px;
    max-width: 90%;
    max-height: 90%;
    overflow: auto;}
	
	.container {
	  max-width: 100%;
	  overflow: auto;
	}
  
	.horizontal-table {
	  display: flex;
	  flex-direction: row;
	  overflow-x: auto;
	  margin-bottom: 10px;
	}
  
	.card {
	  margin: 10px;
	  padding: 10px;
	  border: 1px solid #ccc;
	  border-radius: 5px;
	  min-width: 200px;
	  flex-shrink: 0;
	}
  
	.card-title {
	  font-weight: bold;
	  margin-bottom: 5px;
	}
  
	.job-card {
	  cursor: pointer;
	  background-color: #f8f9fa;
	  padding: 10px;
	  border: 1px solid #ced4da;
	  border-radius: 5px;
	  margin-bottom: 10px;
	}
  
	.job-card:hover {
	  background-color: #e9ecef;
	}
  
	.job-card-selected {
	  background-color: #d4edda;
	}
  
	.job-card-selected:hover {
	  background-color: #c3e6cb;
	}
  
	.candidate-name {
	  cursor: pointer;
	  color: blue;
	  text-decoration: underline;
	}
  
	.modal {
	  position: fixed;
	  top: 0;
	  left: 0;
	  width: 100vw;
	  height: 100vh;
	  display: flex;
	  align-items: center;
	  justify-content: center;
	  z-index: 999;
	  background-color: rgba(0, 0, 0, 0.5);
	}
  
	.modal-content {
	  background-color: #fff;
	  padding: 20px;
	  border-radius: 5px;
	  max-width: 80%;
	  max-height: 80%;
	  overflow: auto;
	}
  
	.candidate-details {
	  margin-top: 10px;
	  padding: 10px;
	  border: 1px solid #ccc;
	  border-radius: 5px;
	}
  </style>
  
  <main class="container mt-4">
	{#if tableVisible}
	<table class="table">
	  <thead class="thead-light">
		<tr>
		  <th>ID</th>
		  <th>Title</th>
		  <th>Reference</th>
		  <th>Status</th>
		  <th>Industry</th>
		  <th>Pipeline</th>
		</tr>
	  </thead>
	  <tbody>
		{#each jsonData as job}
		<tr>
		  <td>{job.id}</td>
		  <td>
			<a href="#" on:click|preventDefault={() => handleTitleClick(job)}>
			  {job.title}
			</a>
		  </td>
		  <td>{job.reference}</td>
		  <td>{job.status}</td>
		  <td>{job.industry}</td>
		  <td>
			<button class="btn btn-primary" on:click|preventDefault={() => handleTitleClick(job)}>View Pipeline</button>
		  </td>
		</tr>
		{/each}
	  </tbody>
	</table>
	{/if}
  </main>
  
  {#if selectedJob}
  <div class="popup">
	<div class="popup-content">
		<h3>Job Pipeline</h3>
	  <div class="horizontal-table">
		<div class="card">
		  <label class="card-title text-primary">Sourced</label>
		</div>
		<div class="card">
		  <label class="card-title text-primary">Applied</label>
		</div>
		<div class="card">
		  <label class="card-title text-primary">CV Shared</label>
		</div>
		<div class="card">
		  <label class="card-title text-primary">Interview</label>
		  {#each idList as job}
		  <div
			class="card job-card {selectedCandidate && selectedCandidate.id === job.id ? 'job-card-selected' : ''}"
			on:click={() => openCandidateDetails(job)}
		  >
			<p>{job.jobRef}</p>
			<p>{job.jobTitle}</p>
			<p class="candidate-name">{job.candidateLabel}</p>
		  </div>
		  {/each}
		</div>
		<div class="card">
		  <label class="card-title text-primary">Shortlist</label>
		</div>
		<div class="card">
		  <label class="card-title text-primary">Offer</label>
		</div>
		<div class="card">
		  <label class="card-title text-primary">Placed</label>
		</div>
	  </div>
	  <button class="btn btn-primary" on:click={() => (selectedJob = null)}>Close</button>
	</div>
  </div>
  {/if}
  
  {#if selectedCandidate}
  <div class="modal">
	<div class="modal-content">
	  <div class="candidate-details">
		<!-- Display candidate details here -->
		<h3>Candidate Details</h3>
		<p>ID: {selectedCandidate.id}</p>
		<p>Job Reference: {selectedCandidate.jobRef}</p>
		<p>Job Title: {selectedCandidate.jobTitle}</p>
		<p>Candidate Label: {selectedCandidate.candidateLabel}</p>
	  </div>
	  <button class="btn btn-primary" on:click={() => (selectedCandidate = null)}>Close</button>
	</div>
  </div>
  {/if}