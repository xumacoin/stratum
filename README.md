coinbase.cpp adds a section commented with `// Xuma //`.

Add this to your stratum source code and rebuild/deploy.

````
	// Xuma //
	if(strcmp(coind->symbol, "XMX") == 0)
	{
		char charity_payee[256] = { 0 };
		const char *payee = json_get_string(json_result, "payee");
		if (payee) snprintf(charity_payee, 255, "%s", payee);
		
		const char* developerfee_payee = json_get_string(json_result, "developerfee_wallet");
		json_int_t developerfee_payment = json_get_int(json_result, "developerfee_amount");
		json_int_t masternode_payment = json_get_int(json_result, "payee_amount");

		strcat(templ->coinb2, "03");
		
		// MASTERNODE
		char script_payee[1024];
		base58_decode(charity_payee, script_payee);
		job_pack_tx(coind, templ->coinb2, masternode_payment, script_payee);
		
		available -= masternode_payment;
		
		// DEVELOPER FEE
		char script_devfee[1024];
		base58_decode(developerfee_payee, script_devfee);
		job_pack_tx(coind, templ->coinb2, developerfee_payment, script_devfee);
		
		available -= developerfee_payment;
		
		// MINER
		job_pack_tx(coind, templ->coinb2, available, NULL);
		strcat(templ->coinb2, "00000000");
		coind->reward = (double)available/100000000*coind->reward_mul;
		
		return;
	}
	// Xuma //
````

Add this to your `yaamp/core/backend/coins.php` file by matching the first existing line in your file and adding the remaining two lines.

````
	$coin->reward = $template['coinbasevalue']/100000000*$coin->reward_mul;

	if($coin->symbol == 'XMX' && isset($template['developerfee_amount']))
		$coin->reward -= $template['developerfee_amount']/100000000;
````

NOTE: Requires version 1.1.2+ of xumad.
