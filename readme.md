Before running repo_setup.sh (Stop vm)

```bash
vagrant halt
```

Add this config to the Vagrantfile

```ruby
config.vm.network :forwarded_port, guest: 3000, host: 3000
config.vm.network :forwarded_port, guest: 3001, host: 3001
config.vm.network :forwarded_port, guest: 3002, host: 3002
config.vm.network :forwarded_port, guest: 3003, host: 3003
config.vm.network :forwarded_port, guest: 3004, host: 3004
config.vm.network :forwarded_port, guest: 3005, host: 3005
config.vm.network :forwarded_port, guest: 3306, host: 3306
```

```bash
vagrant up
```
```bash
mysql -u root < /home/vagrant/apps/stern/db/seeds/sso_dev.sql
```

Add this to a script call it something like database.sh

```bash
bundle exec rails db:environment:set RAILS_ENV=development
bundle exec rake db:drop
bundle exec rake db:setup:versions
bundle exec rake db:setup:payments
bundle exec rake db:setup

bundle exec rake workflow_engine:db:drop
bundle exec rake workflow_engine:db:create
bundle exec rake workflow_engine:db:schema:load
```

Run the script twice database.sh

```bash
bash database.sh
bash database.sh
```

Add this to a script (fixtures.sh)

```bash
bundle exec rake db:load_fixtures FIXTURES=activity_types
bundle exec rake db:load_fixtures FIXTURES=affiliation_details
bundle exec rake db:load_fixtures FIXTURES=affiliation_plans
bundle exec rake db:load_fixtures FIXTURES=affiliation_video_states
bundle exec rake db:load_fixtures FIXTURES=affiliations
bundle exec rake db:load_fixtures FIXTURES=alternate_terms
bundle exec rake db:load_fixtures FIXTURES=api_credentials
bundle exec rake db:load_fixtures FIXTURES=apn_apps
bundle exec rake db:load_fixtures FIXTURES=appointment_methods
bundle exec rake db:load_fixtures FIXTURES=appointment_statuses
bundle exec rake db:load_fixtures FIXTURES=cc_types
bundle exec rake db:load_fixtures FIXTURES=client_focus_categories
bundle exec rake db:load_fixtures FIXTURES=clinical_concepts
bundle exec rake db:load_fixtures FIXTURES=consultation_credit_origination_types
bundle exec rake db:load_fixtures FIXTURES=copay_overrides
bundle exec rake db:load_fixtures FIXTURES=countries
bundle exec rake db:load_fixtures FIXTURES=cpt_codes
bundle exec rake db:load_fixtures FIXTURES=diagnosis_icd9_codes
bundle exec rake db:load_fixtures FIXTURES=doc_types
bundle exec rake db:load_fixtures FIXTURES=emr_systems
bundle exec rake db:load_fixtures FIXTURES=evr_medical_condition_names
bundle exec rake db:load_fixtures FIXTURES=hets_providers
bundle exec rake db:load_fixtures FIXTURES=insurance_payers
bundle exec rake db:load_fixtures FIXTURES=languages
bundle exec rake db:load_fixtures FIXTURES=life_style_conditions
bundle exec rake db:load_fixtures FIXTURES=medical_fields
bundle exec rake db:load_fixtures FIXTURES=medical_fields_physician_types
bundle exec rake db:load_fixtures FIXTURES=medications
bundle exec rake db:load_fixtures FIXTURES=message_types
bundle exec rake db:load_fixtures FIXTURES=mobile_app_versions
bundle exec rake db:load_fixtures FIXTURES=mt_statuses
bundle exec rake db:load_fixtures FIXTURES=mt_types
bundle exec rake db:load_fixtures FIXTURES=new_crop_allergies
bundle exec rake db:load_fixtures FIXTURES=new_crop_dosage_forms
bundle exec rake db:load_fixtures FIXTURES=new_crop_drug_form_and_route_details
bundle exec rake db:load_fixtures FIXTURES=new_crop_drug_mappings
bundle exec rake db:load_fixtures FIXTURES=new_crop_medication_number_types
bundle exec rake db:load_fixtures FIXTURES=new_crop_medications
bundle exec rake db:load_fixtures FIXTURES=note_categories
bundle exec rake db:load_fixtures FIXTURES=note_dispositions
bundle exec rake db:load_fixtures FIXTURES=on_call_statuses
bundle exec rake db:load_fixtures FIXTURES=on_call_types
bundle exec rake db:load_fixtures FIXTURES=org_params
bundle exec rake db:load_fixtures FIXTURES=orgs
bundle exec rake db:load_fixtures FIXTURES=payment_types
bundle exec rake db:load_fixtures FIXTURES=pharmacies
bundle exec rake db:load_fixtures FIXTURES=pharmacy_statuses
bundle exec rake db:load_fixtures FIXTURES=phys_avail_statuses
bundle exec rake db:load_fixtures FIXTURES=phys_on_call_statuses
bundle exec rake db:load_fixtures FIXTURES=physician_types
bundle exec rake db:load_fixtures FIXTURES=plans
bundle exec rake db:load_fixtures FIXTURES=products
bundle exec rake db:load_fixtures FIXTURES=promotional_credit
bundle exec rake db:load_fixtures FIXTURES=rights
bundle exec rake db:load_fixtures FIXTURES=rights_roles
bundle exec rake db:load_fixtures FIXTURES=security_activity_types
bundle exec rake db:load_fixtures FIXTURES=signup_credit
bundle exec rake db:load_fixtures FIXTURES=speciality_certifying_boards
bundle exec rake db:load_fixtures FIXTURES=state_rules
bundle exec rake db:load_fixtures FIXTURES=states
bundle exec rake db:load_fixtures FIXTURES=surgeries
bundle exec rake db:load_fixtures FIXTURES=us_time_zones
bundle exec rake db:load_fixtures FIXTURES=user_statuses
bundle exec rake db:load_fixtures FIXTURES=user_types
bundle exec rake db:load_fixtures FIXTURES=user_widgets
bundle exec rake db:load_fixtures FIXTURES=users
bundle exec rake db:load_fixtures FIXTURES=widget_roles
bundle exec rake db:load_fixtures FIXTURES=widgets
```

Execute the script (fixtures.sh)
```bash
bash fixtures.sh
```

After that add this to script (db.sh), add the following content

```bash
bundle exec rake mdlive:create_patients
bundle exec rake mdlive:create_ca_patients
bundle exec rake mdlive:create_providers
bundle exec rake mdlive:create_ca_providers
bundle exec rake mdlive:create_csa_agents
bundle exec rake roles:add_roles_to_csa
bundle exec rake roles:enable_csa_access_for_old_csa_roles
bundle exec rails runner "Provider.all.each {|p| ProviderProfile.create(user_id: p.id, ped_from_age: 3, ped_to_age: 15) }"
```

Run the script 
```bash
bash db.sh
```

After that enter the database and run the following content

```bash
INSERT INTO `user_types` (`name`, `can_admin_system`, `can_admin_physicians`, `can_see_patients`, `can_admin_cs_reps`, `can_help_customers`, `can_admin_sales_reps`, `can_process_sales`, `can_manage_account`, `can_fulfill`, `can_fulfill_pharmacy`, `can_admin_pharmacy`, `can_message`, `start_page`, `created_at`, `updated_at`, `can_super_admin`, `csa_enabled`)
VALUES
	('Workflow Admin', 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 'customer_service', '2022-06-10 10:56:28', '2022-06-10 10:56:28', 0, 1);

```

After that copy the content of suffix.rb and paste it the rails console inside stern.

After that run in the same rails console

```bash
Provider.all.each {|p| p.create_provider_detail!(billing_suffix: "MD") if p.provider_detail.blank?}
```

Add the following alias in the .bashrc

```bash
alias stern='cd /home/vagrant/apps/stern && bundle exec rails s -p 3001 -b 0.0.0.0'
alias pportal='cd /home/vagrant/apps/provider-portal && bundle exec rails s -p 3005 -b 0.0.0.0'
```

```bash
create database telehealth_development;
create database versions_development;
create database workflow_engine_development;

drop database payments;
drop database sso_dev;
drop database telehealth_development;
drop database versions_development;
drop database workflow_engine_development;
```

Finally go to browser in http://localhost:3005/login

```bash
User: demoprovider1 
Password: mdlive123
```
