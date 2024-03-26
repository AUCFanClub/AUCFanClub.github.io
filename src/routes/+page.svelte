<script>
	let AUC_target = 500;
	let CrCl = 90;
	let Weight = 70;
	let VD_factor = "0.65";
	let calc_type = "empiric";
	let k = 0;
	let Vd = 0;
	let est_dose_24 = 0;
	let est_dose = 0;
	let est_tau = 0;
	let selected_dose = 0;
	let selected_interval = 0;
	let selected_infusion_time = 1;
	let AUC_SS = 0;
	let peak_SS = 0;
	let trough_SS = 0;

	function calc_dose() {
		est_dose_24 = k * Vd * AUC_target;
		est_tau = (Math.log(35/12.5)/k)+1;
		est_dose = est_dose_24 / (24/est_tau);
	}

	function calc_SS() {
		AUC_SS = (selected_dose/(k*Vd))*(24/selected_interval);
		let temp1 = (selected_dose / selected_infusion_time)/(k*Vd);
		let temp2 = (1-Math.exp(-k*selected_infusion_time));
		let temp3 = (1-Math.exp(-k*selected_interval));
		peak_SS = temp1 * (temp2/temp3);
		trough_SS = peak_SS * Math.exp(-k*(selected_interval - selected_infusion_time));
	}

	$: if (CrCl > 0) {
		k = 0.00083 * CrCl + 0.0044;
		calc_dose();
	}

	$: if (Weight > 0) {
		Vd = Weight * parseFloat(VD_factor);
		calc_dose();
	}

	$: if (selected_dose > 0 && selected_interval > 0) {
		calc_SS();
	}
	else {
		AUC_SS = 0;
	}
</script>

<div class="container">
	<h1>AUC Fan Club - Vancomycin Calculator</h1>
	<form>
		<div class="input-group mb-3">
			<select bind:value={calc_type} class="form-select" aria-label="Default select example">
				<option selected value="empiric">Calculate an empiric regimen</option>
				<option value="two_levels_ss">Two levels at STEADY STATE</option>
				<option value="two_levels_first">Two levels after FIRST dose</option>
				<option value="single_level">Single level at steady state</option>
			</select>
		</div>
		<div class="input-group mb-3">
			<div class="input-group-prepend">
				<span class="input-group-text" id="basic-addon1">AUC Target</span>
			</div>
			<input bind:value={AUC_target} type="number" class="form-control" />
		</div>
		<div class="input-group mb-3">
			<div class="input-group-prepend">
				<span class="input-group-text" id="basic-addon1">CrCl</span>
			</div>
			<input bind:value={CrCl} type="number" class="form-control" />
			<div class="input-group-append">
				<span class="input-group-text">ml/min</span>
			</div>
		</div>
		<div class="input-group mb-3">
			<div class="input-group-prepend">
				<span class="input-group-text" id="basic-addon1">Weight</span>
			</div>

			<input bind:value={Weight} type="number" class="form-control" />
			<div class="input-group-append">
				<span class="input-group-text">kg</span>
			</div>
		</div>
		<div class="input-group mb-3">
			<div class="input-group-prepend">
				<span class="input-group-text" id="basic-addon1">Volume of Distribution</span>
			</div>
			<select bind:value={VD_factor} class="form-select" aria-label="Default select example">
				<option selected value="0.65">Normal (0.65)</option>
				<option value="0.5">Obese (0.5)</option>
				<option value="0.6">Dehydrated (0.6)</option>
				<option value="0.7">Overhydrated (0.7)</option>
				<option value="0.7">Septic Shock/ICU/Trauma (0.7)</option>
				<option value="0.7">Pregnant in 3rd Trimester or less than 48h post partum (0.7)</option>
			</select>
			<div class="input-group-append">
				<span class="input-group-text">L/kg</span>
			</div>
		</div>
	<p>k = {k.toFixed(4)}</p>
	<p>half life = {(Math.log(2) / k).toFixed(2)} hours</p>
	<p>Vd = {Vd.toFixed(2)} L</p>
	<p>Recommended total daily dose: {est_dose_24.toFixed(0)} mg</p>
	<p>Recommended maintenance dose: {est_dose.toFixed(0)} mg</p>
	<p>Recommended interval: {est_tau.toFixed()} hours</p>
		<div class="input-group mb-3">

			<div class="input-group-prepend">
				<span class="input-group-text" id="basic-addon1">Dose</span>
			</div>
			<input bind:value={selected_dose} type="number" class="form-control" />
			<div class="input-group-append">
				<span class="input-group-text">mg</span>
			</div>
		</div>
		<div class="input-group mb-3">
			<div class="input-group-prepend">
				<span class="input-group-text" id="basic-addon1">Interval</span>
			</div>

			<input bind:value={selected_interval} type="number" class="form-control" />
			<div class="input-group-append">
				<span class="input-group-text">hours</span>
			</div>
		</div>
		<div class="input-group mb-3">
			<div class="input-group-prepend">
				<span class="input-group-text" id="basic-addon1">Infusion Time</span>
			</div>

			<input bind:value={selected_infusion_time} type="number" class="form-control" />
			<div class="input-group-append">
				<span class="input-group-text">hours</span>
			</div>
		</div>
		{#if AUC_SS > 0}
		<h3>Projected Steady State Values:</h3>
		<p>AUC: {AUC_SS.toFixed(0)}</p>
		<p>Peak: {peak_SS.toFixed(1)}</p>
		<p>Trough: {trough_SS.toFixed(1)}</p>
		{/if}
	</form>
</div>
