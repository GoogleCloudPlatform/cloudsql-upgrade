<!-- Copyright 2024 Google LLC

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License. -->

New Cloud SQL upgrade tool for MySQL & PostgreSQL major versions and Enterprise Plus


Upgrading to a newer major version of a database can be complex, time-consuming and error-prone. At Google Cloud, we want to make sure that your database upgrades go smoothly, without compromising the security or availability of your applications. We also aim to minimize the risks and disruptions associated with upgrades, to save you time and resources.

As a fully-managed database service, Cloud SQL supports multiple major versions of MySQL and PostgreSQL, each of which introduces performance improvements, security enhancements and expanded feature sets. Today, we’re excited to announce an automated Cloud SQL upgrade tool for major versions and Enterprise Plus, available for MySQL 5.7 to 8.0.31, and PostgreSQL 9.6, 10, and 11 to PostgreSQL 14 and 15. The tool provides automated upgrade assessments, scripts to resolve issues and in-place major version upgrades, as well as Enterprise Plus edition upgrades, all in one go. 

It’s particularly useful for organizations that want to avoid extended support fees associated with Cloud SQL extended support for MySQL and PostgreSQL end-of-life versions, or that want to take advantage of the performance and availability features of Enterprise Plus editions.


Key features 
The Cloud SQL upgrade tool provides: 

Automated pre-upgrade assessment, where checks are curated based on recommendations available for MySQL and PostgreSQL upgrades, as well as from insights from real customer experiences.

Detailed assessment reports

Automated scripts to resolve issues

In-place major version and Enterprise Plus upgrades leveraging Cloud SQL’s in-place major version upgrade feature.

You can visualize how these elements work together in the diagram below.

https://storage.googleapis.com/gweb-cloudblog-publish/images/1-_Architecture_Rz3TUpk.max-1600x1600.png


Using the Cloud SQL upgrade tool 
Upgrading an instance with the Cloud SQL upgrade tool is straightforward:

1. Execute the binary file on a VM that has connectivity to the target Cloud SQL instance. This runs an upgrade assessment, generates reports along with automated scripts to fix identified issues, and creates a clone of the Cloud SQL instance.

2. Verify the assessment report and apply any necessary manual fixes on the cloned instance.

3. Re-execute the binary to ensure there are no pending issues.

4. Once verified, confirm that you agree to continue with the in-place major version or Enterprise Plus upgrade.

5. Run tests on the upgraded instance to verify that the application functions correctly. 

6. Upon successful testing, execute automated fix scripts and any necessary manual fixes on the production instance, run the assessment, verify the report, and finally, initiate in-place major version upgrade. 

If you are interested in the tool,  reach out to your Google representative.