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
  <img src="docs/img/6.1P2.PNG" width="900">
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
  <img src="docs/img/6.1P3.PNG" width="900">
</p>

<p><b>Exam Insight:</b> If your plays take too long on “Gathering Facts,” check SSH connectivity and name resolution first.</p>

<hr/>

<h2>6.4 Using Multi-Tier Variables</h2>
<p>Ansible merges variables from multiple levels (global → group → host → task). Facts live near the top in precedence order.</p>
<ul>
  <li>Playbook vars override gathered facts only temporarily.</li>
  <li>Fact caching can persist facts between runs (if configured).</li>
</ul>

<hr/>

<h2>6.5 Dictionaries and Arrays</h2>
<p>Facts are stored as nested dictionaries (YAML = JSON). Access keys with dot or bracket notation.</p>

<pre><code>{{ ansible_facts['default_ipv4']['address'] }}
{{ ansible_facts.os_family }}
</code></pre>

<p><b>Why:</b> Needed when looping or templating dynamic data from hosts.</p>

<hr/>

<h2>6.6 Defining Custom Facts</h2>
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

<h2>6.7 Variable Precedence Refresher</h2>
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
  <li>Facts make plays adaptive — vital for targeting, templating, and conditional logic.</li>
</ul>

<i>End of Lesson 6 notes</i>
