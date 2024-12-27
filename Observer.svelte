<script>
	import { onMount, afterUpdate } from 'svelte';
  
	export let observers = []; // Array of objects: { beforeClass, afterClass, once }
  
	let intersectionObservers = [];
  
	const initializeObservers = () => {
	  // Disconnect existing observers to prevent duplicates on re-render or navigation
	  intersectionObservers.forEach((obs) => obs.disconnect());
	  intersectionObservers = [];
  
	  // Create a new observer for each configuration object
	  observers.forEach(({ beforeClass, afterClass, once }) => {
		const observer = new IntersectionObserver((entries) => {
		  entries.forEach((entry) => {
			if (entry?.isIntersecting) {
			  entry.target.classList.add(afterClass);
			  if (once) {
				observer.unobserve(entry.target); // Unobserve if "once" is true
			  } else {
				entry.target.classList.remove(beforeClass); // Remove hidden class for re-observation
			  }
			} else if (!once) {
			  entry.target.classList.add(beforeClass); // Reapply hidden class if out of view
			}
		  });
		});
  
		// Observe elements with the specified "beforeClass"
		const elements = document.querySelectorAll(`.${beforeClass}`);
		elements.forEach((el) => observer.observe(el));
  
		intersectionObservers.push(observer); // Keep track of the observer
	  });
	};
  
	// Initialize observers when the component is mounted
	onMount(() => {
	  initializeObservers();
  
	  return () => {
		// Cleanup on unmount
		intersectionObservers.forEach((obs) => obs.disconnect());
		intersectionObservers = [];
	  };
	});
  
	// Reinitialize observers when "observers" or DOM changes (e.g., after navigation)
	afterUpdate(() => {
	  initializeObservers();
	});
</script>

<!--make the compiler not steal the fucking visible class stfu mf-->
{#each [...new Set(observers.map(o => o.afterClass))] as afterClass}
  <!-- svelte-ignore element_invalid_self_closing_tag -->
  <div id="dummy-{afterClass}" class={afterClass} style="display: none;" />
{/each}
