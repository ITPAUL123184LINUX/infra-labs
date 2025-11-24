<h1 align="center">Lesson 9: Understanding Ansible Roles</h1>

<p>
Ansible <strong>roles</strong> are the foundation of reusable, organized automation. When your playbooks start growing large and repetitive, roles let you break them down into clean, modular components. This approach promotes consistency, readability, and collaboration across teams and environments.
</p>

<hr>

<h2>What Are Roles?</h2>

<p>
Roles are Ansible’s way of organizing automation into smaller, reusable units of work. They allow you to structure tasks, variables, templates, files, and handlers in a predictable directory format. Think of roles as "mini playbooks" designed for specific system functions — such as installing Apache, managing users, or hardening SSH.
</p>

<ul>
  <li>Roles are about <strong>reusable code</strong> — build once, reuse many times.</li>
  <li>They provide <strong>standardized solutions</strong> for common automation tasks.</li>
  <li>Community roles are shared publicly through <a href="https://galaxy.ansible.com">Ansible Galaxy</a>.</li>
  <li>Custom roles can be stored in local Git repositories or internal automation hubs.</li>
  <li>Roles are included in playbooks using the <code>roles:</code> section under a play header.</li>
</ul>

<hr>

<h2>Why Roles Matter</h2>

<p>
As your automation footprint expands, so does the need for maintainability. Roles let you:
</p>

<ul>
  <li>Separate logic by purpose (e.g., <code>apache</code>, <code>users</code>, <code>selinux</code>).</li>
  <li>Reuse tested components across multiple playbooks.</li>
  <li>Reduce human error by following predictable patterns.</li>
  <li>Standardize deployments across environments and teams.</li>
</ul>

<p>
This structure turns your automation into a scalable codebase — not just a collection of ad hoc playbooks.
</p>

<hr>

<h2>Basic Role Structure</h2>

<p>
A role follows a specific directory layout that Ansible automatically recognizes:
</p>

<pre><code>
roles/
└── apache/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    ├── files/
    ├── vars/
    │   └── main.yml
    ├── defaults/
    │   └── main.yml
    ├── meta/
    │   └── main.yml
</code></pre>

<p>
Each subdirectory serves a specific purpose:
</p>

<ul>
  <li><strong>tasks/</strong> – Defines what the role does (main logic).</li>
  <li><strong>handlers/</strong> – Contains service restart/reload logic.</li>
  <li><strong>templates/</strong> – Jinja2 templates rendered dynamically.</li>
  <li><strong>files/</strong> – Static files copied to managed nodes.</li>
  <li><strong>vars/</strong> – Variables with high precedence (rarely changed).</li>
  <li><strong>defaults/</strong> – Variables with lowest precedence (user-overridable).</li>
  <li><strong>meta/</strong> – Role dependencies and metadata.</li>
</ul>

<hr>

<h2>Using a Role in a Playbook</h2>

<p>
To use a role, reference it inside a play under the <code>roles:</code> key:
</p>

<pre><code class="language-yaml">
- name: Configure Apache Web Server
  hosts: webservers
  become: yes
  roles:
    - apache
</code></pre>

<p>
When the playbook runs, Ansible automatically looks for the <code>apache</code> role inside the <code>roles/</code> directory and executes its <code>tasks/main.yml</code> file.
</p>

<hr>

<h2>Creating a Role</h2>

<p>
You can quickly generate a new role using the <code>ansible-galaxy</code> command:
</p>

<pre><code class="language-bash">
ansible-galaxy init roles/firewall
</code></pre>

<p>
This command creates the full directory structure with placeholder files. You can then define your logic inside <code>tasks/main.yml</code> and progressively build from there.
</p>

<hr>

<h2>Example: Simple Role Implementation</h2>

<pre><code class="language-yaml">
# roles/apache/tasks/main.yml
- name: Install Apache
  ansible.builtin.dnf:
    name: httpd
    state: present

- name: Ensure Apache is running and enabled
  ansible.builtin.service:
    name: httpd
    state: started
    enabled: yes
</code></pre>

<p>
Then, include this role in a playbook:
</p>

<pre><code class="language-yaml">
- name: Deploy web servers using Apache role
  hosts: webservers
  become: yes
  roles:
    - apache
</code></pre>

<p>
This approach eliminates redundancy — the same Apache setup logic can now be reused in multiple playbooks across environments.
</p>

<hr>

<h2>Best Practices for Roles</h2>

<ul>
  <li>Keep roles focused on a single responsibility (e.g., one service or function per role).</li>
  <li>Use <strong>defaults/main.yml</strong> for tunable variables that users may override.</li>
  <li>Document variable usage and expected inputs clearly in each role’s README.</li>
  <li>Store custom roles in a version-controlled Git repository for reuse and tracking.</li>
  <li>Test roles independently before integrating them into larger playbooks.</li>
  <li>Leverage Ansible Galaxy to import trusted community roles and learn best practices.</li>
</ul>

<hr>

<h2>Summary</h2>

<p>
Roles make Ansible scalable. They enforce structure, encourage reusability, and prevent duplication. As projects mature, roles become the natural next step to evolve from procedural automation toward modular, enterprise-ready infrastructure as code.
</p>

<p>
In the next section, we’ll explore how <strong>role dependencies, variables, and defaults</strong> interact — and how to structure advanced playbooks that seamlessly integrate multiple roles into a single automated workflow.
</p>

<hr>
<h1 align="center">Lesson 9 Continued: Using Galaxy Roles</h1>

<p>
Community roles expand Ansible’s power by allowing engineers to use prebuilt automation shared by the global community. These roles are hosted on <a href="https://galaxy.ansible.com">Ansible Galaxy</a> and can be easily searched, installed, and integrated into your environment.
</p>

<h2>Installing Galaxy Roles</h2>

<p>
To install a community role from Ansible Galaxy, use:
</p>

<pre><code class="language-bash">
ansible-galaxy role install geerlingguy.apache
</code></pre>

<p>
This command downloads the specified role and places it in your local <code>roles_path</code>. The <strong>roles_path</strong> defines where Ansible looks for roles. The default search order is:
</p>

<ul>
  <li><code>roles/</code> directory in the current project</li>
  <li><code>~/.ansible/roles/</code></li>
  <li><code>/etc/ansible/roles/</code></li>
  <li><code>/usr/share/ansible/roles/</code></li>
</ul>

<p>
You can install a role in a custom location using the <code>-p</code> flag:
</p>

<pre><code class="language-bash">
ansible-galaxy role install -p ./custom_roles geerlingguy.apache
</code></pre>

<p>
This flexibility allows teams to organize roles per project or environment while maintaining a consistent search order.
</p>

<hr>

<h2>Using the ansible-galaxy Command</h2>

<p>
The <code>ansible-galaxy</code> command provides several key functions for managing roles. Below are common examples:
</p>

<pre><code class="language-bash">
# Search for roles related to Docker by author geerlingguy for RHEL
ansible-galaxy role search 'docker' --author geerlingguy --platforms EL

# Get detailed info about a role
ansible-galaxy role info geerlingguy.docker

# List all installed roles
ansible-galaxy role list

# Remove a role
ansible-galaxy role remove geerlingguy.docker
</code></pre>

<p>
These commands streamline discovery, management, and cleanup of roles in your automation environment. They are especially useful when auditing what’s currently deployed or verifying version compatibility across multiple systems.
</p>

<hr>

<h2>Using a Requirements File</h2>

<p>
When your project depends on multiple roles, manually installing each one becomes inefficient. Instead, define all dependencies inside a <code>roles/requirements.yml</code> file. This creates a single source of truth for managing project-level role dependencies.
</p>

<pre><code class="language-yaml">
# roles/requirements.yml
- src: https://github.com/mygit/myrole
  scm: git
- src: file:///tmp/myrole.tar
- src: https://www.example.local/myrole.tar
</code></pre>

<p>
You can install all roles listed in the requirements file with one command:
</p>

<pre><code class="language-bash">
ansible-galaxy role install -r roles/requirements.yml
</code></pre>

<p>
This ensures consistency across environments — every team member or build pipeline installs the same versions from the same sources. It’s also ideal for CI/CD automation and reproducible infrastructure setups.
</p>

<h2>Sample Requirements File</h2>

<pre><code class="language-yaml">
- src: https://github.com/myaccount/myrole.role
  scm: git
  version: "2.0"
- src: file:///tmp/myrole.tar
  name: mytarrole
- src: https://example.local/myrole.tar
  name: mywebrole
</code></pre>

<p>
Roles can be sourced from Git, files, or web URLs. When hosted in Git, the <code>scm: git</code> key is required to specify the source control mechanism.
</p>

<hr>

<h2>Practical Use Case</h2>

<p>
Consider a scenario where your team manages both web and database servers. Instead of creating all automation from scratch, you can pull stable, community-tested roles for Apache and PostgreSQL, then extend them with custom logic in your own local roles.
</p>

<pre><code class="language-yaml">
- name: Deploy web and database stack
  hosts: all
  become: yes
  roles:
    - geerlingguy.apache
    - geerlingguy.postgresql
    - custom.firewall
</code></pre>

<p>
This hybrid approach leverages both community expertise and your organization’s specific compliance requirements.
</p>

<hr>

<h2>Best Practices</h2>

<ul>
  <li>Pin versions in <code>requirements.yml</code> for deterministic builds.</li>
  <li>Keep community roles separate from custom roles to avoid accidental overwrites.</li>
  <li>Review and audit community roles before production use for security and compliance.</li>
  <li>Document role usage, purpose, and required variables in each project’s README.</li>
</ul>

<hr>

<h2>Summary</h2>

<p>
Using Galaxy roles and the <code>ansible-galaxy</code> command unlocks collaboration and standardization across teams. Combined with <code>requirements.yml</code>, this approach enforces consistency, speeds up onboarding, and keeps automation modular and clean.
</p>

<p>
Next, we’ll dive into how to <strong>build role dependencies, integrate roles with collections, and automate role testing</strong> — skills that demonstrate advanced Ansible maturity and are crucial for real-world production automation.
</p>

<hr>
<h1 align="center">Lesson 9 Continued: Creating Custom Roles</h1>

<p>
Beyond community roles, many organizations require <strong>custom roles</strong> to meet internal standards or environment-specific configurations. Custom roles let you define automation logic tailored to your exact infrastructure, security, and compliance needs.
</p>

<h2>Creating a Custom Role</h2>

<p>
To create a new role manually, ensure it resides within your <code>roles_path</code> — this is where Ansible searches for role definitions. You can verify or modify this path inside <code>ansible.cfg</code>.
</p>

<p>
To automatically generate the required role structure, use:
</p>

<pre><code class="language-bash">
ansible-galaxy role init myrole
</code></pre>

<p>
This command creates the default folder hierarchy for <code>myrole</code> inside the current <code>roles_path</code>. Once created, complete the <code>tasks/main.yml</code> file and other directories as needed.
</p>

<pre><code class="language-yaml">
# roles/myrole/tasks/main.yml
- name: Example custom task
  ansible.builtin.debug:
    msg: "My custom role is working!"
</code></pre>

<p>
Custom roles follow the same structure and execution model as community roles, making them easy to integrate with existing playbooks.
</p>

<hr>

<h1 align="center">Lesson 9 Continued: Installing RHEL System Roles</h1>

<p>
Red Hat Enterprise Linux includes a powerful suite of prebuilt automation content called <strong>RHEL System Roles</strong>. These roles standardize configuration management across RHEL versions and simplify administration tasks such as networking, storage, time synchronization, and security.
</p>

<h2>Installing RHEL System Roles</h2>

<ul>
  <li>For Red Hat customers, the roles are distributed as the <strong>redhat.rhel_system_roles</strong> collection.</li>
  <li>For Ansible Core users, they’re available via the <code>rhel-system-roles</code> RPM package.</li>
</ul>

<pre><code class="language-bash">
sudo dnf install -y rhel-system-roles
</code></pre>

<p>
Installing this package places roles under:
</p>

<ul>
  <li><code>/usr/share/ansible/roles/</code></li>
  <li><code>/usr/share/ansible/collections/</code></li>
</ul>

<p>
This ensures easy access via both <code>ansible-playbook</code> and <code>ansible-navigator</code>.  
The package also installs examples under <code>/usr/share/doc/rhel-system-roles/</code> — a great resource for learning or adapting configurations quickly.
</p>

<p><strong>Tip:</strong> Best practice is to install using <code>dnf install rhel-system-roles</code> to ensure dependencies and documentation are properly placed.</p>

<hr>

<h2>Understanding RHEL System Roles</h2>

<p>
RHEL System Roles provide a unified interface to manage configuration parameters across different Red Hat versions. They help maintain consistent configurations, even as system defaults evolve between RHEL 7, 8, and 9.
</p>

<pre><code class="language-bash">
sudo dnf install -y rhel-system-roles
</code></pre>

<p>
Once installed, explore available roles:
</p>

<pre><code class="language-bash">
ls /usr/share/ansible/roles/rhel-system-roles.*
</code></pre>

<p>
Example directories include:
<ul>
  <li><code>rhel-system-roles.network</code></li>
  <li><code>rhel-system-roles.timesync</code></li>
  <li><code>rhel-system-roles.selinux</code></li>
</ul>
</p>

<p>
Each role includes sample playbooks under <code>/usr/share/doc/rhel-system-roles/</code>, providing a quick start for automation.
</p>

<hr>

<h2>Example: Using a RHEL System Role</h2>

<p>
Here’s how to apply the <strong>timesync</strong> role to configure NTP across managed hosts:
</p>

<pre><code class="language-yaml">
- name: Configure NTP using RHEL system role
  hosts: all
  become: yes
  roles:
    - rhel-system-roles.timesync
</code></pre>

<p>
Variables such as <code>timesync_ntp_servers</code> can be set in your playbook or inventory to define preferred time sources.
</p>

<hr>

<h2>Exam Tip</h2>

<p>
If you encounter RHEL System Roles during certification exams (like RHCE), you can copy a sample playbook directly from:
</p>

<pre><code class="language-bash">
/usr/share/doc/rhel-system-roles/
</code></pre>

<p>
Then, adapt it to your needs rather than building one from scratch. This saves time and ensures accuracy under exam conditions.
</p>

<hr>

<h2>Summary</h2>

<p>
RHEL System Roles bridge the gap between manual configuration and enterprise automation. They simplify complex tasks and ensure that automation scripts remain compatible across RHEL versions. Combined with custom roles, they empower system administrators to manage configurations efficiently, consistently, and at scale.
</p>

<p>
In the next section, we’ll explore <strong>Role Dependencies, Collections, and Testing Roles</strong> to finalize our understanding of modular, production-ready Ansible automation.
</p>


