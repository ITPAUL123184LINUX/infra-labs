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
<ul>
  <li>Facts are key-value data that describe each host (OS, network, CPU, memory, etc.).</li>
  <li>They’re gathered automatically at the start of a play via <code>setup</code> module.</li>
  <li>They live in <code>ansible_facts</code> — a large dictionary available to every task.</li>
  <li>You can disable gathering with <code>gather_facts: no</code> for faster runs.</li>
</ul>

<pre><code>ansible all -m ansible.builtin.setup | head -n 25</code></pre>

<p><b>Shortcut:</b> See a single fact:</p>
<pre><code>ansible all -m ansible.builtin.setup -a 'filter=ansible_hostname' -o</code></pre>

<hr/>

<h2>6.2 Using Multi-Tier Variables</h2>
<p>Ansible merges variables from multiple levels (global → group → host → task). Facts live near the top in precedence order.</p>
<ul>
  <li>Playbook vars override gathered facts only temporarily.</li>
  <li>Fact caching can persist facts between runs (if configured).</li>
</ul>

<hr/>

<h2>6.3 Dictionaries and Arrays</h2>
<p>Facts are stored as nested dictionaries (YAML = JSON). Access keys with dot or bracket notation.</p>

<pre><code>{{ ansible_facts['default_ipv4']['address'] }}
{{ ansible_facts.os_family }}
</code></pre>

<p><b>Why:</b> Needed when looping or templating dynamic data from hosts.</p>

<hr/>

<h2>6.4 Defining Custom Facts</h2>
<ul>
  <li>Create files under <code>/etc/ansible/facts.d/</code> on managed nodes.</li>
  <li>Each file must end with <code>.fact</code> and contain INI or JSON data.</li>
  <li>Gathered automatically during <code>setup</code>.</li>
</ul>

<pre><code>[localinfo]
site=datacenter1
tier=web
</code></pre>

<pre><code>ansible all -m ansible.builtin.setup -a 'filter=ansible_local' -o</code></pre>

<p><b>Result:</b> Accessible via <code>{{ ansible_local.localinfo.site }}</code></p>

<hr/>

<h2>6.5 Variable Precedence Refresher</h2>
<ul>
  <li>Most specific wins.</li>
  <li>Order: <b>CLI -e</b> → <b>Play vars</b> → <b>Host vars</b> → <b>Group vars</b> → <b>Facts</b> → <b>Defaults</b>.</li>
  <li>Facts can be overridden but only in memory — not permanently.</li>
</ul>

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

<p><b>Expected:</b> Each managed node prints its own data and saves a local JSON fact file.</p>

<p align="center">
  <img src="docs/img/lesson6-facts-lab.png" alt="Lesson 6 facts lab output" width="900"/>
</p>

<hr/>

<h2>Exam Focus Tips</h2>
<ul>
  <li>Know how to gather and filter facts with <code>setup</code>.</li>
  <li>Recognize <code>ansible_facts</code> as read-only host data.</li>
  <li>Define and reference <code>ansible_local</code> custom facts.</li>
  <li>Understand variable precedence order.</li>
  <li>Facts help make plays dynamic — vital for configuration targeting and conditionals.</li>
</ul>

<i>End of Lesson 6 notes</i>

