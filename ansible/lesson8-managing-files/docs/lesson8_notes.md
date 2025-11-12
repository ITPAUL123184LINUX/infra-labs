<h1 align="center">Lesson 8: File Management Modules</h1>

<p>
Ansible provides several <strong>file management modules</strong> that allow administrators to manipulate files and directories in a predictable, idempotent way. These are essential for maintaining configuration files, enforcing policies, and collecting file-level data across systems. Below is a breakdown of the core modules with clear explanations and usage guidance.
</p>

<hr>

<h2>🔹 1. ansible.builtin.lineinfile</h2>

<p>
<strong>Purpose:</strong> Ensures that a specific line exists in a file. Commonly used for modifying single-line configurations (e.g., updating <code>/etc/ssh/sshd_config</code>).
</p>

<pre><code class="language-yaml">
- name: Ensure a line exists in /etc/motd
  ansible.builtin.lineinfile:
    path: /etc/motd
    line: "Authorized access only."
</code></pre>

<p>
<strong>Explanation:</strong>  
This module scans the file and inserts or updates the line only if needed — maintaining idempotency. Perfect for enforcing individual parameters like “PermitRootLogin no”.
</p>

<hr>

<h2>🔹 2. ansible.builtin.blockinfile</h2>

<p>
<strong>Purpose:</strong> Manages <em>multi-line</em> text blocks in files. Useful for inserting configuration sections rather than individual lines.
</p>

<pre><code class="language-yaml">
- name: Insert a configuration block in /etc/ssh/sshd_config
  ansible.builtin.blockinfile:
    path: /etc/ssh/sshd_config
    block: |
      Match Group admins
        PasswordAuthentication no
</code></pre>

<p>
<strong>Explanation:</strong>  
This module places a block of text between marker lines. It is ideal when you need to manage grouped settings or templates that span multiple lines.
</p>

<hr>

<h2>🔹 3. ansible.builtin.file</h2>

<p>
<strong>Purpose:</strong> Sets file attributes such as ownership, permissions, and type. It can also create directories, remove files, and handle symbolic links.
</p>

<pre><code class="language-yaml">
- name: Create a directory with specific permissions
  ansible.builtin.file:
    path: /var/log/custom
    state: directory
    owner: root
    group: root
    mode: '0755'
</code></pre>

<p>
<strong>Explanation:</strong>  
This module is the backbone of file system management. Whether it’s creating directories, touching files, or managing links — <code>file</code> ensures system consistency.
</p>

<hr>

<h2>🔹 4. ansible.builtin.stat</h2>

<p>
<strong>Purpose:</strong> Collects file metadata — permissions, size, checksum, and more. When combined with <code>register</code>, it becomes a powerful validation tool.
</p>

<pre><code class="language-yaml">
- name: Get file details
  ansible.builtin.stat:
    path: /etc/hosts
  register: hostfile_info

- name: Display file size
  debug:
    msg: "The /etc/hosts file size is {{ hostfile_info.stat.size }} bytes."
</code></pre>

<p>
<strong>Explanation:</strong>  
Use this when you need to verify file existence or attributes before proceeding with another task. For example, check if a file exists before overwriting it.
</p>

<hr>

<h2>Practical Example</h2>

<pre><code class="language-yaml">
- name: Configure SSH and verify file changes
  hosts: all
  become: yes
  tasks:
    - name: Ensure root login is disabled
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PermitRootLogin'
        line: 'PermitRootLogin no'

    - name: Add a security block for admins
      ansible.builtin.blockinfile:
        path: /etc/ssh/sshd_config
        block: |
          Match Group admins
            PasswordAuthentication no

    - name: Ensure log directory exists
      ansible.builtin.file:
        path: /var/log/custom
        state: directory
        owner: root
        group: root
        mode: '0755'

    - name: Verify /etc/ssh/sshd_config permissions
      ansible.builtin.stat:
        path: /etc/ssh/sshd_config
      register: ssh_stat

    - name: Display file mode
      debug:
        msg: "Current permissions: {{ ssh_stat.stat.mode }}"
</code></pre>

<h2>📘 Summary</h2>

<ul>
  <li><strong>lineinfile</strong> → Best for single-line edits.</li>
  <li><strong>blockinfile</strong> → Best for multi-line configuration blocks.</li>
  <li><strong>file</strong> → Controls file and directory attributes.</li>
  <li><strong>stat</strong> → Gathers metadata for validation and conditional logic.</li>
</ul>

<p>
Together, these modules provide complete control over file content and structure in Ansible automation. Mastering them is essential for configuration management and achieving idempotent, secure, and repeatable deployments.
</p>

<h1 align="center">File Copying Modules</h1>

<p>
While the previous section focused on editing and managing file content, this section covers how to <strong>copy, transfer, and synchronize</strong> files between nodes — a critical skill for configuration deployment and data management.
</p>

<hr>

<h2>🔹 5. ansible.builtin.copy</h2>

<p>
<strong>Purpose:</strong> Copies a file from the control node to a managed host. Often used to deploy configuration files, scripts, or templates.
</p>

<pre><code class="language-yaml">
- name: Copy a configuration file to remote server
  ansible.builtin.copy:
    src: files/httpd.conf
    dest: /etc/httpd/conf/httpd.conf
    owner: root
    group: root
    mode: '0644'
</code></pre>

<p>
<strong>Explanation:</strong>  
If the destination file already exists and differs, it will be replaced only if the content is different — maintaining idempotency. Excellent for pushing files consistently across nodes.
</p>

<hr>

<h2>🔹 6. ansible.builtin.fetch</h2>

<p>
<strong>Purpose:</strong> Retrieves files <em>from</em> managed hosts back to the control node. Useful for gathering logs, reports, or backups.
</p>

<pre><code class="language-yaml">
- name: Fetch remote log file to control node
  ansible.builtin.fetch:
    src: /var/log/messages
    dest: ./logs/
    flat: yes
</code></pre>

<p>
<strong>Explanation:</strong>  
It copies files in the opposite direction of <code>copy</code>. The <code>flat</code> option prevents directory nesting under the hostname.
</p>

<hr>

<h2>🔹 7. ansible.posix.synchronize</h2>

<p>
<strong>Purpose:</strong> Synchronizes files between systems using the <code>rsync</code> protocol. Requires <code>rsync</code> to be installed on both control and managed hosts.
</p>

<pre><code class="language-yaml">
- name: Synchronize web content to remote server
  ansible.posix.synchronize:
    src: /var/www/html/
    dest: /var/www/html/
    delete: yes
</code></pre>

<p>
<strong>Explanation:</strong>  
Efficient for mirroring directories or maintaining backups. It transfers only changed data, minimizing bandwidth and time.
</p>

<hr>

<h2>🔹 8. ansible.posix.patch</h2>

<p>
<strong>Purpose:</strong> Applies patches to existing files using diff/patch logic.
</p>

<pre><code class="language-yaml">
- name: Apply a security patch to /etc/ssh/sshd_config
  ansible.posix.patch:
    src: patch/sshd.patch
    dest: /etc/ssh/sshd_config
</code></pre>

<p>
<strong>Explanation:</strong>  
Used primarily for automated code or config updates across environments — a safe and trackable way to apply controlled modifications.
</p>

<hr>

<h2>📘 Summary: File Copying Modules</h2>

<ul>
  <li><strong>copy</strong> → Push files <em>to</em> managed hosts.</li>
  <li><strong>fetch</strong> → Pull files <em>from</em> managed hosts.</li>
  <li><strong>synchronize</strong> → Mirror directories efficiently using rsync.</li>
  <li><strong>patch</strong> → Apply diff patches safely and consistently.</li>
</ul>

<p>
Together, these modules enable robust file transfer and synchronization workflows in Ansible. They’re critical for maintaining consistency between environments, automating deployments, and ensuring every managed system matches the desired configuration state.
</p>

<hr>
<h1 align="center">Understanding Jinja2 Templates</h1>

<p>
When <code>lineinfile</code> and <code>blockinfile</code> reach their limits for dynamic or variable-driven configuration, Jinja2 templates become the next-level solution. Jinja2 is a powerful templating engine for Python that allows you to generate configuration files dynamically based on variables defined within playbooks, inventory, or facts.
</p>

<h2>Jinja2 Templates Overview</h2>

<ul>
  <li><strong>lineinfile</strong> and <strong>blockinfile</strong> handle simple text changes.</li>
  <li>For more advanced, variable-based configurations, use <strong>Jinja2 templates</strong>.</li>
  <li>Templates are rendered by the <code>ansible.builtin.template</code> module.</li>
  <li>Variables inside templates are enclosed in <code>{{ }}</code> and are replaced with actual values at runtime.</li>
</ul>

<pre><code class="language-yaml">
- name: Deploy Apache configuration using a template
  ansible.builtin.template:
    src: templates/httpd.conf.j2
    dest: /etc/httpd/conf/httpd.conf
    owner: root
    group: root
    mode: '0644'
</code></pre>

<p>
Example <strong>httpd.conf.j2</strong> template:
</p>

<pre><code>
ServerAdmin {{ admin_email }}
Listen {{ http_port }}
DocumentRoot {{ document_root }}
</code></pre>

<p>
The variables <code>admin_email</code>, <code>http_port</code>, and <code>document_root</code> can be defined in your playbook or inventory files. During playbook execution, Ansible renders this template into a complete configuration file with actual values.
</p>

<p><strong>Best Practice:</strong> Use Jinja2 templates when files differ per host or environment and depend on variable data. For simpler static edits, <code>lineinfile</code> and <code>blockinfile</code> are faster and lighter.</p>

<p>
<strong>Best Practice:</strong> Always include an <code>ansible_managed</code> header in critical configuration templates. It creates traceability, discourages manual edits, and reinforces operational discipline in automated environments.
</p>

<hr>
<h1 align="center">Using for Statements in Jinja2 Templates</h1>

<p>
In Jinja2 templates, <strong>for statements</strong> allow you to iterate over lists, dictionaries, or host groups dynamically. This makes templates far more flexible, enabling Ansible to generate different configurations for multiple users, hosts, or environments in a single template.
</p>

<h2>Loop Syntax</h2>

<p>
A Jinja2 loop begins with <code>{% for %}</code> and ends with <code>{% endfor %}</code>. Inside the loop, you can reference each item using a variable name.
</p>

<pre><code>
{% for user in users %}
  {{ user }}
{% endfor %}

{% for host in groups['webservers'] %}
  {{ host }}
{% endfor %}
</code></pre>

<p>
In this example:
<ul>
  <li><code>users</code> could be a list of usernames defined in your inventory or playbook.</li>
  <li><code>groups['webservers']</code> dynamically loops through all hosts in the “webservers” inventory group.</li>
</ul>
</p>

<p>
When rendered, Jinja2 will replace each variable with its actual value, generating one line of output per loop iteration.
</p>

<h2>Example: Creating a User Configuration File</h2>

<pre><code class="language-yaml">
- name: Create a users configuration file dynamically
  ansible.builtin.template:
    src: templates/users.conf.j2
    dest: /etc/app/users.conf
</code></pre>

<p>
<strong>users.conf.j2 Template Example:</strong>
</p>

<pre><code>
# Managed by Ansible
{% for user in users %}
User: {{ user }}
{% endfor %}
</code></pre>

<p>
If your playbook defines:
</p>

<pre><code class="language-yaml">
vars:
  users:
    - alice
    - bob
    - carol
</code></pre>

<p>
Then the rendered file will look like this:
</p>

<pre><code>
# Managed by Ansible
User: alice
User: bob
User: carol
</code></pre>

<h2>Best Practice and Exam Tip</h2>

<p>
Jinja2 <strong>loops</strong> and <strong>conditionals</strong> (<code>if</code>, <code>elif</code>, <code>else</code>) are heavily used in real-world Ansible automation and are frequently tested in exams. They allow for dynamic file generation based on data, inventory, or host-specific variables. Mastering these ensures your templates remain scalable and adaptable across environments.
</p>

<p>
<strong>Tip:</strong> Practice combining <code>for</code> loops with conditionals (e.g., only include certain items if a variable matches). This deepens your understanding and helps you build templates that respond intelligently to context.
</p>

<hr>
<h1 align="center">Using Modules to Manage SELinux</h1>

<p>
Ansible provides specific modules for managing <strong>SELinux (Security-Enhanced Linux)</strong> contexts and policies. Understanding the correct module for each use case is essential to maintain compliance and avoid conflicts between temporary file changes and permanent policy updates.
</p>

<h2>Key SELinux Modules</h2>

<ul>
  <li><strong>ansible.builtin.file</strong>: Can set SELinux file context using the <code>seuser</code>, <code>serole</code>, <code>setype</code>, and <code>selevel</code> parameters. However, this changes only the file label directly—similar to using the <code>chcon</code> command—and is not persistent across relabel operations.</li>

  <li><strong>community.general.sefcontext</strong>: Safely manages SELinux file context rules within the SELinux policy itself (persistent changes). This is the correct method to make sure your changes survive relabeling or reboots.</li>

  <li><strong>ansible.builtin.command</strong>: Often used to trigger <code>restorecon</code> after modifying SELinux contexts. This re-applies the policy-defined contexts to the filesystem.</li>
</ul>

<h2>Example: Defining and Applying Persistent SELinux Contexts</h2>

<pre><code class="language-yaml">
- name: Manage SELinux file contexts for web content
  hosts: webservers
  become: yes
  tasks:
    - name: Add SELinux context rule for /webdata directory
      community.general.sefcontext:
        target: '/webdata(/.*)?'
        setype: httpd_sys_content_t
        state: present

    - name: Apply SELinux context changes
      ansible.builtin.command:
        cmd: restorecon -Rv /webdata
</code></pre>

<p>
In this example:
<ul>
  <li><code>sefcontext</code> adds or updates the policy rule so that files under <code>/webdata</code> always receive the correct SELinux label (<code>httpd_sys_content_t</code>).</li>
  <li><code>restorecon</code> reapplies the correct contexts to existing files, ensuring consistency.</li>
</ul>
</p>

<h2>Important Notes</h2>

<ul>
  <li>Do <strong>not</strong> rely on <code>ansible.builtin.file</code> to set SELinux context for long-term configurations—it mimics <code>chcon</code>, which is temporary and overridden by relabels.</li>
  <li>Persistent policy changes must be handled through <code>community.general.sefcontext</code>.</li>
  <li>Always follow context changes with a <code>restorecon</code> operation to enforce consistency.</li>
  <li>For enterprise-scale automation, Red Hat recommends using the <strong>RHEL System Role for SELinux</strong> for more comprehensive policy control.</li>
</ul>

<p>
By using these modules together, you ensure that SELinux remains both enforced and predictable across all managed systems — a crucial aspect of compliance, hardening, and security automation.
</p>



