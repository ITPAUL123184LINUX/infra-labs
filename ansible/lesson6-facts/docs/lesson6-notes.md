<h1>Lesson 6 – Working with Facts (Deep Notes)</h1>

<h2>Goal</h2>
<b>Learn how Ansible gathers, uses, and defines system information (“facts”) to build dynamic automation.</b>

<h3>Control node</h3>
<ul>
  <li><b>Host:</b> <code>control</code> (where Ansible commands run)</li>
</ul>

<h3>Managed nodes</h3>
<ul>
  <li><b>Hosts:</b> <code>ansible1</code>, <code>ansible2</code>, <code>ansible3</code> — reachable via SSH as user <b>ansible</b>.</li>
</ul>

<hr/>

<h2>6.1 Understanding Facts</h2>
<p>
Ansible <b>facts</b> are automatically gathered variables that describe a host’s system information.  
They’re discovered when a play begins using the <code>setup</code> module.
</p>

<ul>
  <li>Facts contain host data like memory, IP, OS, and hardware details.</li>
  <li>Stored in a dictionary called <code>ansible_facts</code>.</li>
  <li>Useful for making plays dynamic (no hardcoded values).</li>
  <li>Examples: <code>ansible_facts.default_ipv4.address</code>, <code>ansible_facts.memtotal_mb</code>, <code>ansible_facts.distribution</code>.</li>
</ul>

<p><b>Example:</b></p>
<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.1ipfacts.yml.PNG"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.1ipfacts-output.PNG">
</p>

<hr/>

<h2>6.2 Managing Fact Gathering</h2>

<ul>
  <li>By default, Ansible gathers facts automatically before running tasks.</li>
  <li>Disable it to speed things up:
    <pre><code>gather_facts: no</code></pre>
  </li>
  <li>Even if disabled, you can re-gather with the <code>setup</code> module:
    <pre><code>ansible all -m ansible.builtin.setup -a 'filter=ansible_hostname' -o</code></pre>
  </li>
  <li>View all host data with:
    <pre><code>ansible all -m ansible.builtin.setup | head -n 25</code></pre>
  </li>
  <li>Facts follow a dotted hierarchy — like <code>ansible_facts.default_ipv4.address</code> — meaning you can drill into nested data.</li>
</ul>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.1P2.PNG" width="900">
</p>

<p><b>Tip:</b> Use <code>debug</code> to display any single fact for clarity:</p>
<pre><code>- debug:
    msg: "OS family is {{ ansible_facts.os_family }}"
</code></pre>

<hr/>

<h2>6.3 Troubleshooting Slow Fact Collection</h2>

<ul>
  <li>Fact gathering can slow down large environments or misconfigured networks.</li>
  <li>Solutions:
    <ul>
      <li>Enable <b>fact caching</b> to reuse facts between runs.</li>
      <li>Ensure proper <b>hostname resolution</b> — each host must resolve the others.</li>
      <li>Maintain consistent <code>/etc/hosts</code> entries across all managed nodes.</li>
    </ul>
  </li>
</ul>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.1P3.PNG" width="900">
</p>

<p><b>Exam Insight:</b> If your plays hang on “Gathering Facts,” check SSH and DNS first.</p>

<hr/>

<h2>6.4 Using Multi-Tier Variables</h2>
<p>Ansible merges variables from multiple levels (global → group → host → play → task). Facts live high in this hierarchy.</p>
<ul>
  <li>Playbook vars override gathered facts only during that play.</li>
  <li>Fact caching keeps facts persistent across runs when configured.</li>
</ul>

<hr/>

<h2>6.5 Dictionaries and Arrays (Deep Dive)</h2>
<p>
Multi-valued variables can be <b>lists</b> (arrays) or <b>dictionaries</b> (key-value hashes).  
Facts themselves are dictionaries, while lists are ideal for loops.
</p>

<ul>
  <li><b>Dictionary:</b> <code>{ }</code> unordered key-value pairs — like <code>ansible_facts</code>.</li>
  <li><b>List:</b> <code>[ ]</code> ordered items — used with <code>loop:</code>.</li>
  <li><b>String:</b> Plain quoted text (<code>" "</code>).</li>
</ul>

<p><b>Dictionary Example:</b></p>
<pre><code>users:
  linda:
    shell: /bin/bash
  lisa:
    shell: /bin/sh
</code></pre>

<p>or:</p>
<pre><code>users:
  linda: { shell: '/bin/bash' }
  lisa:  { shell: '/bin/sh' }
</code></pre>

<p><b>Accessing values:</b></p>
<pre><code>{{ users.linda.shell }}
{{ users['lisa']['shell'] }}
</code></pre>

<hr/>

<h2>Lesson 6 Lab – Dictionaries vs Arrays</h2>
<p>Compare how Ansible handles both structures:</p>

<ul>
  <li><b>Lists:</b> Great for looping over many items.</li>
  <li><b>Dictionaries:</b> Best for named, structured data (like host facts).</li>
</ul>

<hr/>

<h2>6.6 Defining Custom Facts</h2>
<ul>
  <li>Create custom data in <code>/etc/ansible/facts.d/</code> on managed hosts.</li>
  <li>Files must end in <code>.fact</code> and contain INI or JSON data.</li>
  <li>Automatically gathered by the <code>setup</code> module.</li>
</ul>

<pre><code>[localinfo]
site=datacenter1
tier=web
</code></pre>

<pre><code>ansible all -m ansible.builtin.setup -a 'filter=ansible_local' -o</code></pre>

<p><b>Result:</b> Access via <code>{{ ansible_local.localinfo.site }}</code></p>

<hr/>

<h2>Lesson 6 Lab – Working with Facts</h2>

<pre><code>---
- name: demo facts
  hosts: all
  tasks:
    - name: print hostname and IP
      ansible.builtin.debug:
        msg: "Host {{ ansible_hostname }} has IP {{ ansible_default_ipv4.address }}"

    - name: save facts to file
      ansible.builtin.copy:
        dest: "/tmp/{{ inventory_hostname }}_facts.txt"
        content: "{{ ansible_facts | to_nice_json }}"
</code></pre>

<p><b>Expected:</b> Each node prints its hostname/IP and saves its own fact file under <code>/tmp</code>.</p>

<hr/>

<h2>6.7 Review Summary</h2>
<ul>
  <li>Facts = system details gathered by <code>setup</code>.</li>
  <li>Facts live inside <code>ansible_facts</code> as dictionaries.</li>
  <li>Lists (<code>[ ]</code>) are ordered — use for loops.</li>
  <li>Dictionaries (<code>{ }</code>) are named — use for structured data.</li>
</ul>

<hr/>

<h2>6.8 Understanding and Using Custom Facts (Screenshots Summary)</h2>

<p>
Custom facts let you label each host with extra info Ansible can read automatically.  
They live <b>on the managed node</b> inside <code>/etc/ansible/facts.d/</code> and are gathered as part of <code>ansible_facts.ansible_local</code>.
</p>

<ul>
  <li><b>Why they matter:</b> They bridge static data and dynamic logic. Example — tagging servers by role, environment, or state.</li>
  <li><b>File format:</b> INI or JSON, must end with <code>.fact</code>.</li>
  <li><b>Each section</b> inside the file needs a label (e.g., <code>[software]</code>).</li>
</ul>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.4.PNG" width="850"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.4newlocalfacts.yml.PNG" width="850"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.4newlocalfacts.ymloutpt.PNG" width="850"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.4 localfacts-verify.PNG" width="850"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson6-facts/docs/img/6.4p2.PNG" width="850">
</p>

<h3>Example – Installing and Verifying Custom Facts</h3>

<pre><code>---
- name: install custom facts
  hosts: lamp
  vars:
    remote_dir: /etc/ansible/facts.d
    facts_file: localfacts.fact
  tasks:
    - name: create fact directory
      ansible.builtin.file:
        path: "{{ remote_dir }}"
        state: directory
        recurse: yes
    - name: install new facts
      ansible.builtin.copy:
        src: "{{ facts_file }}"
        dest: "{{ remote_dir }}"
</code></pre>

<h3>Verification Command</h3>

<pre><code>ansible ansible2 -m ansible.builtin.setup -a "filter=ansible_local"
</code></pre>

<p><b>Output (simplified):</b></p>
<pre><code>"ansible_local": {
  "localfacts": {
    "software": {
      "package": "httpd",
      "service": "httpd",
      "state": "enabled"
    }
  }
}
</code></pre>

<h3>Key Takeaways</h3>
<ul>
  <li>Custom facts are stored under <code>ansible_facts.ansible_local</code>.</li>
  <li>Use <code>ansible hostname -m setup -a "filter=ansible_local"</code> to display them.</li>
  <li>Reference them with <code>{{ ansible_facts.ansible_local.localfacts.software.state }}</code>.</li>
  <li>Think of them as per-host metadata that travels with the machine itself.</li>
</ul>

<p><b>Simplified Insight:</b>  
Host variables live on the control node. Custom facts live on the host.  
When you gather facts, Ansible pulls both — but custom facts are like tattoos on each server describing its true identity.</p>

<hr/>

<h2>Exam Focus Tips</h2>
<ul>
  <li>Memorize variable precedence and where facts fit in.</li>
  <li>Know how to gather, filter, and display both system and custom facts.</li>
  <li>Understand the difference between <code>ansible_facts.local</code> and <code>ansible_facts.ansible_local</code>.</li>
  <li>Be ready to show or use custom facts dynamically in playbooks.</li>
</ul>

<i>End of Lesson 6 notes (final version)</i>
