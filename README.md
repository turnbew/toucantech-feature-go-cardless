FOR PRIVACY AND CODE PROTECTING REASONS THIS IS A SIMPLIFIED VERSION OF CHANGES AND NEW FEATURES

TASK DATE: 08.05.2018 - FINISHED: 29.05.2018

TASK LEVEL: ADVANCED (HIGH)

TASK SHORT DESCRIPTION: [Setting/integrating new payment types into TT system VIA API calls: GOCARDLES]

GITHUB REPOSITORY CODE: feature/task-1575-ordering-records-in-groups

NOTE:  
	- DOCUMENTATION: https://developer.gocardless.com/api-reference/	
	
PHASIS IN WORK:
	- DONE: API communication between TT site and GoCardless
	- DONE: NEW class for hangling GoCardless API calls
	- DONE: New Error handling
	- DONE: CREATING new GoCardles mini form onto TT system (4 forms in one)
	- DONE: NEW technic of input field checkin
	- DONE: NO CACHING AT GOCARDLES MINIFORM - modal - IS DONE - moving all content of loaded gocardless mini form
	- DONE: if using saved account and pay once ->checking not working properly
	- DONE: Account holder's name needed - 
	- DONE: IF changing on-off payment - just that allowed
	- DONE: IF monthly then just that .... etc
	- DONE: Checkin input fields is not working properly
	- DONE: changing static test for lang variable in GC mini form
	- DONE: change in DB default_fundraising_payments - payment_processor_slug - if gocardless then not paypal
	- DONE: using or not test bank details
	- DONE: error messages handling
	- DONE: set up payments and subscriptions = scheduled payments
	- DONE: closing colorbox and modal after setting up direct debit
	- DONE: set up GoCardless payment at front_cart and events as well
	- DONE: managing webhooks	

USING GOCARDLES
	STEP I. Install GoCardless API third-party
	
CHANGES
 
	NEW FILES AND FOLDERS 
	
		NEW THIRD PARTY FOLDER: libraries\gocardless-sdk
		controllers\gocardless.php
		english\go_cardless_lang.php
		models\go_cardless_m.php
		gateway_forms\gocardless.php
 
	IN FILES: 
	
		partials\payment_form.php
		
			ADDED CODE 
			
				inside JS script tag
					
					case 'gocardless':
						$('#myGoCardlessPaymentModal').appendTo("body").modal('show');
					break;
					
				before JS script tag 
				
					<?php echo $goCardlessModal; ?>
		
			CHANGED CODE
			
				TO: 
					<?php $gatewaysForms  = $goCardlessModal = ''; ?>
					<?php foreach ($gateways_form_data as $gateway => $form_data): ?>
						<?php
							if ($gateway != 'gocardless') {
								$gatewaysForms .= $this->load->view('../../network_settings/views/general_settings/partials/gateway_forms/' . $gateway, $form_data, true);
							}
							else {
								$goCardlessModal = $this->load->view('../../network_settings/views/general_settings/partials/gateway_forms/' . $gateway, $form_data, true);
							}
						?>
					<?php endforeach; ?>
					
					<div id="gateway-forms-container" style="display: none;">
						<?php echo $gatewaysForms; ?>
					</div>
			
			
			
	
		views\supportus.php
	
			ADDED CODE:
			
				{{ asset:js file="bootstrap.js" group="static" }}
	
	
		
	
	
	controllers\front_cart.php
	
			ADDED CODE: 
			
				Inside function 

					case 'gocardless' : 
						$userId = $this->current_user ? $this->current_user->id : null;
					
						if ( ! is_null($userId) ) {
							#loading profile_m model and gocardless controlle
							$this->load->model('bbusers/profile_m');
							$this->load->controller('gocardless', 'network_settings');	
							
							#getting user's details - we'll need go_cardless_customer_id
							$user = $this->profile_m->get_user_info($userId);						
							
							#we try to grabbing user's bank account details (just with basic details) from GoCardless account  
							$bankAccounts = array();
							if ( trim($user->go_cardless_customer_id, "/r/n") != '' ) {
								#getting GoCardless client
								$gcClient = $this->gocardless->apiGetGcClient($paymentAccount->id);
								
								#getting GoCardless user's bank accounts
								if ( ! is_null($gcClient) ) {
									if ( $this->gocardless->apiExistGcCustomer($user->go_cardless_customer_id) ) {
										$bankAccounts = $this->gocardless->apiGetGcBankAccounts($user->go_cardless_customer_id);
									}
								}
							}
							
							#setting payment method to the payments drop down list
							$paymentMethods[] = array(
								..............
							);
							
							#setting details to the GoCardles form
							$gatewaysFormData['gocardless'] = array(
								'modalTitle'		=> 'Total price ',
								'userId'            => $userId,
								..............
							);
						}
					break;	

				
	
	
		controllers\bbevents.php
		
			ADDED CODE: 
			
				Inside function register_reservation
		
					case 'gocardless' : 
						$userId = $registrations_summary['primary']['user_id'] ? $registrations_summary['primary']['user_id'] : null;
					
						if ( ! is_null($userId) ) {
							#loading profile_m model and gocardless controlle
							$this->load->model('bbusers/profile_m');
							$this->load->controller('gocardless', 'network_settings');	
							
							#getting user's details - we'll need go_cardless_customer_id
							$user = $this->profile_m->get_user_info($userId);						
							
							#we try to grabbing user's bank account details (just with basic details) from GoCardless account  
							$bankAccounts = array();
							if ( trim($user->go_cardless_customer_id, "/r/n") != '' ) {
								#getting GoCardless client
								$gcClient = $this->gocardless->apiGetGcClient($payment_account->id);
								
								#getting GoCardless user's bank accounts
								if ( ! is_null($gcClient) ) {
									if ( $this->gocardless->apiExistGcCustomer($user->go_cardless_customer_id) ) {
										$bankAccounts = $this->gocardless->apiGetGcBankAccounts($user->go_cardless_customer_id);
									}
								}
							}
							
							#setting payment method to the payments drop down list
							$paymentMethods[] = array(
								..............
							);
							
							#setting details to the GoCardles form
							$gateways_form_data['gocardless'] = array(
								'modalTitle'		=> 'Tickets for ' . $registrations_summary['event']->title . ' ',
								'userId'            => $userId,
								..............
							);
						}
					break;	
	
	
	
	
		\network-site\addons\default\modules\fundraising\controllers\fundraising.php
	
			ADDED CODE: 
			
				Inside function getHTMLForPaymentMethods
				
					case 'gocardless' : 
							#loading profile_m model and gocardless controlle
							$this->load->model('bbusers/profile_m');
							$this->load->controller('gocardless', 'network_settings');	
							
							#getting user's details - we'll need go_cardless_customer_id
							$user = $this->profile_m->get_user_info($settings['user_id']);						
							
							#we try to grabbing user's bank account details (just with basic details) from GoCardless account  
							$bankAccounts = array();
							if ( trim($user->go_cardless_customer_id, "/r/n") != '' ) {
								#getting GoCardless client
								$gcClient = $this->gocardless->apiGetGcClient($paymentAccount->id);
								
								#getting GoCardless user's bank accounts
								if ( ! is_null($gcClient) ) {
									if ( $this->gocardless->apiExistGcCustomer($user->go_cardless_customer_id) ) {
										$bankAccounts = $this->gocardless->apiGetGcBankAccounts($user->go_cardless_customer_id);
									}
								}
							}
							
							#setting payment method to the payments drop down list
							$paymentMethods[] = array(
								..............
							);
							
							#setting details to the GoCardles form
							$recurringType = strtolower($settings['payment_recurring_type']);
							$recurringType = (($recurringType == 'm') ? 'monthly' : ( 
												($recurringType == 'w') ? 'weekly' : (
													($recurringType == 'y') ? 'yearly' : ''))); 
							$gatewaysFormData['gocardless'] = array(
								'modalTitle'		=> 'Donate to ' . $settings['campaign_name'],
								'userId'            => $settings['user_id'],
								..............
							);
						break;		
	
		
		
		config\routes.php
		
			ADDED CODE:
			
				/* GoCardless webhook redirection */
				$route['............../event-listener'] = '............../webhookEventListener';
	
	
	
	
		general_settings\payment_accounts_form.php
	
			CHANGED CODE 
			
				FROM: 
					<!--Gateway settings templates-->
					<div id="gateway-settings-templates" style="display:none;">
					<?php foreach ($gateways as $slug => $gateway): ?>
					<div id="<?php echo $slug; ?>-gateway-settings">
						<?php if ( ! empty($gateway['settings'])): ?>
							<?php foreach ($gateway['settings'] as $name => $label): ?>
								<li>
									<label for="<?php echo $name; ?>"><?php echo $label; ?></label>
									<div class="input">
										<?php
											$form_input_function = ($name === 'password') ? 'form_password' : 'form_input';
											
											echo $form_input_function("settings[$name]", set_value("settings[$name]", empty($account->settings[$name]) ? null : $account->settings[$name]));
										?>
									</div>
								</li>
							<?php endforeach; ?>
						<?php endif; ?>
					</div>
					<?php endforeach; ?>
					</div>
							
			
				TO:
					<!--Gateway settings templates-->
					<div id="gateway-settings-templates" style="display:none;">
					<?php foreach ($gateways as $slug => $gateway): ?>
					<div id="<?php echo $slug; ?>-gateway-settings">
						<?php if ( ! empty($gateway['settings'])): ?>
							<?php foreach ($gateway['settings'] as $name => $label): ?>
								<?php if ( ! is_array($label) ) { ?>
								<li>
									<label for="<?php echo $name; ?>"><?php echo $label; ?></label>
									<div class="input">
										<?php
											$form_input_function = ($name === 'password') ? 'form_password' : 'form_input';
											
											echo $form_input_function("settings[$name]", set_value("settings[$name]", empty($account->settings[$name]) ? null : $account->settings[$name]));
										?>
									</div>
								</li>
								<?php } else { ?>
								<li>
									<label><?php echo $label['label']; ?></label>
									<div class="input">
										<?php echo $label['note']; ?>
									</div>
								</li>	
								<?php } ?>
							<?php endforeach; ?>
						<?php endif; ?>
					</div>
					<?php endforeach; ?>
					</div>
	
	
	
	
		
		models\payment_accounts_m.php
	
			ADDED CODE:
			
				Inside GLOBAL VARIABLE  protected $gateways
				
					'gocardless' => array( 
						'name' => 'GoCardless',
						'settings' => array(
							'api_access_token' => 'API access token',
							'account_type' => 'Account type (live or sandbox)',
						),
					),
					
				Inside public function __construct and sections_supported_gateways array
				
					In all of the cases: 'gocardless',
	
				Inside public function __construct as well
				
					$this->gateways['gocardless'] = array( 
						'name' => 'GoCardless',
						'settings' => array(
							'api_access_token' => 'API access token',
							'account_type' => 'Account type (live or sandbox)',
							'webhook_name' => array(
								'label' => 'WebHook name in your GoCardless account (optional)', 
								'note' => 'GoCardless event listener'
							),
							'webhook_url' => array(
								'label' => 'WebHook URL in your GoCardless account', 
								'note' => site_url() . 'go-cardless/event-listener'
							),
							'webhook_secret' => array(
								'label' => 'WebHook secret key in your GoCardless account', 
								'note' => 'Use your API access token above'
							),
						),
					);
	
	
	
	
		Services\BaseService.php
		
			CHANGED CODE: 
			
				Inside function __construct
	
					FROM:  return $body->data;
					
					TO:  return (isset($body->data)) ? $body->data : '';
						

	
	
		Resources\BaseResource.php
		
			CHANGED CODE: 
			
				Inside function __construct
	
					FROM: 
						foreach($data as $key => $value) {
							if (property_exists(get_class($this), $key)) {
								$this->{$key} = $value;
							}
						}
					
					TO: 
						if (is_object($data)) {
							if ( count($data) > 0) {
								foreach($data as $key => $value) {
									if (property_exists(get_class($this), $key)) {
										$this->{$key} = $value;
									}
								}
							}
						}
					
					
		Core\ApiClient.php
		
			CHANGED CODE: 
			
				Inside handleErrors function
				
					FROM: $api_response = new ApiResponse($response);
						  throw new $exception_class($api_response);
				
					TO:  return $api_response = new ApiResponse($response);
						//throw new $exception_class($api_response);
	
	
	
	
		libraries\Options.php
		
			Added New Function:

				public function countryCodes($code = '', $countryName = '', $defaultCode = 'GB', $defaultCountry = 'United Kingdom') 
				{
					$codes = array (
						'AF' => 'Afghanistan', 
						....
						'ZW' => 'Zimbabwe',
					);		
					
					if ( $code != '' ) {
						return ( isset($codes[$code]) ) ? $codes[$code] : $defaultCode;
					}
					else if ( $countryName != '' ) {
						$key = array_search ($countryName, $codes);
						return ( ! $key ) ? $defaultCountry : $key; 
					}
					else {
						return $codes;
					}
				} //END function countryCodes		
			
				
				
		
		bbusers\details.php
		
			ADDED CODE
				
				Inside Upgrade function 
				
					// Add GoCardless Customer_ID
					if (version_compare($old_version, '2.4.27', 'lt')) {
						if ( ! $this->modify_profiles_stream_nb() ) {
							$this->session->set_flashdata('notice','<b>Update Notice:</b> Failed to run modify_profiles_stream_nb');
							return false;
						}
					}
		
			CHANGED CODE 
			
				Inside modify_profiles_stream_nb
				
					..... # this part is added
					// Stripe Customer_ID
					$fields[] = array(
						'name'          => 'GoCardless Customer_ID',
						'slug'          => 'go_cardless_details',
						'type'          => 'text',
						'required'      => false
					);
						
					.... #this changed
					
					$table = $this->db->dbprefix('data_fields');
		
					foreach ($fields as $field) 
					{
						$field['assign'] = 'profiles';
						$field['namespace'] = 'users';
						
						if ( ! $this->db->value_exists($field['slug'], 'field_slug', $table, 'field_namespace = "' . $field['namespace'] . '"'))  {
							if ( $this->streams->fields->add_field($field) === false ) return false;
						}
					}
					
					....

				
