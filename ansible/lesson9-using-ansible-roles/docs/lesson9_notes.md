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
