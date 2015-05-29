# Whipping Up Awesome
## TDD with [Test Kitchen](http://kitchen.ci)
### [@cheeseplus](http://twitter.com/cheeseplus)
### [cheeseplus@chef.io](mailto:cheeseplus@chef.io)

---

#### This is Fletcher Nichol. He is awesome.
##### And now he is one of us <3

![Fletcher Rocking Out](images/fletcher-nichol.jpg)

---

``` > whoami```

---

#### cheeseplus
##### (Seth Thomas)

* Sandwich Arist @ Subway
* Sales Associate (CPUs) @ CompUSA (RIP)
* Senior Student Associate @ UT ITS HelpDesk
* Student Sysadmin @ UT Recsports
* Build Master @ Sony Online Entertainment (RIP)
* Directory Services Lead (OpenLDAP) @ UT IDM Team
* Sysadmin @ Optaros
* Build/Operations Engineer @ Vigil Games - THQ (RIP)
* Ops Engineer -> Evangelist @ Basho
* Partner Engineer -> Global Evangelist @ Chef

Note: 
Started very young - Dad is a massive nerd and we had computers in the home early
Dad was also into robotics and amateur rocketry so I was encouraged very young
He was a stay at home dad as well - He is awesome
We were a military family so we moved around a lot, as such the internet kept us connected to families/friends
I was writing Basic games on a Commodore64 at the age of 8
"You'll never make any money playing video games" -> 14 years later I won the bet 

---

### Strong Opinions
#### <span style="color:#42affa">Or how I learned to stop worrying and love Illumos</span>

* ZFS is the best file system
* DTrace is the best system tracing tool
* SMF is the best service init/manager
* We owe Sun a ton for doing amazing R&D <3
* Brendan Gregg is a wizard

---

### Strong Opinions
#### <span style="color:#42affa">Humans are more important than computers</span>

* Empathy Driven Development 
* Community > Technical Perfection
* Use the right tool for the right job
* Examine all assumptions critically
* Fundamentals are IMPORTANT
* Telemetry > Measuring everything
* Remember that humans make software. They have feelings.

Note:
Empathy and community is one of the biggest reasons I started using Chef
It's also why I work here
Inspirations for growing knowledge:
Brendan Gregg (Joyent, Netflix), Theo Schlossnagle (OmniTI), Scott Fritchie, 
a dozen (ex) Bashos, Kyle Kingsbury, Camille Fournier, Adam Jacob, Noah Kantrowitz,
Miah Johnson, Matt Suave-Frankel, Joshua Timberman, and dozens of others

---

### Why?

If you are writing a cookbook you want to test it before putting it into production. Test-kitchen provides a framework that, in conjunction with the tools I will describe, helps enable test driven development (TDD)

---

#### Test Kitchen allows one to

* creation of compute resources for multiple platforms (concurrently)
* converge cookbooks based on platform + suite defintions
* integration test the created compute resources
* destroy compute resources when done

---

#### Platform configuration


<span style="color:#42affa">.kitchen.yml</span>
```yaml
platforms:
  - name: ubuntu-14.04
  - name: centos-7.0
  - name: centos-6.6
```

---

#### Driver configuration

<span style="color:#42affa">.kitchen.yml</span>

```yaml
---
driver:
  name: vagrant
  network:
    - ["private_network", {ip: "192.168.11.333"}]

provisioner:
  name: chef_zero
  chef_version: 12.3.0
```

---

#### Driver configuration (cloud)

<span style="color:#42affa">.kitchen.cloud.yml</span>

<span style="color:#42affa">export KITCHEN_YAML=.kitchen.$WHATEVS.yml</span>

```yaml
---
driver:
  name: ec2
  aws_access_key_id: <%= ENV["AWS_ACCESS_KEY_ID"] %>
  aws_secret_access_key: <%= ENV["AWS_SECRET_ACCESS_KEY"] %>
  aws_ssh_key_id: <%= ENV["AWS_KEYPAIR_NAME"] %>

platforms:
- name: amazon-2014.03.1
  driver:
    image_id: ami-ed8e9284
    username: ec2-user
    ssh_key: <%= ENV["EC2_SSH_KEY_PATH"] %>
 ```

---

### Suite configuration

```yaml
suites:
  - name: coprhd-host-add
    run_list:
      - recipe[coprhd_host_test::add]
    attributes:
      coprhd:
        user: <%= ENV['VIPR_USER'] %>
        password:  <%= ENV['VIPR_PASS'] %>
        url: <%= ENV['VIPR_URL'] %>
        host:
          type: 'Linux'
          ip_or_dns: '192.168.10.111'
          user_name: 'root'
          password:  'password'
```

---

#### [Foodcritic](http://www.foodcritic.io/)

![Foodcritic](images/foodcritic.png)

* A linter and style tool for cookbooks


---

#### [Rubocop](https://github.com/bbatsov/rubocop)

![Rubocop](images/rubocop.png)

* A Ruby static code analyzer
* Based on [Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide)

---

### Rubocop configuration


<span style="color:#42affa">.rubocop.yml</span>
```yaml
suites:
  - name: coprhd-host-add
    run_list:
      - recipe[coprhd_host_test::add]
    attributes:
      coprhd:
        user: <%= ENV['VIPR_USER'] %>
        password:  <%= ENV['VIPR_PASS'] %>
        url: <%= ENV['VIPR_URL'] %>
        host:
          type: 'Linux'
          ip_or_dns: '192.168.10.111'
          user_name: 'root'
          password:  'password'
```

---

### [TravisCI](https://travis-ci.org/)

* Continous testing and deployment
* Free for Open Source projects
* Integrates with GitHub [pull requests](https://github.com/chef-cookbooks/mysql/pull/343)

Note:
Who all uses TravisCI?

---

#### TravisCI Configuration


<span style="color:#42affa">.travis.yml</span>
```yaml
language: ruby
bundler_args: --without kitchen_vagrant
rvm:
- 2.1.0
script:
- rake spec
```

---

#### Rake
##### Alternative - Thor

<span style="color:#42affa">Rakefile</span>
```ruby
# Rspec and ChefSpec
desc 'Run ChefSpec examples'
RSpec::Core::RakeTask.new(:spec)

# Integration tests. Kitchen.ci
namespace :integration do
  desc 'Run Test Kitchen with Vagrant'
  task :vagrant do
    Kitchen.logger = Kitchen.default_file_logger
    Kitchen::Config.new.instances.each do |instance|
      instance.test(:always)
    end
  end
```

---

#### Guardfile
##### <span style="color:#42affa">Extra Credit</span>

```ruby
guard 'rubocop' do
  watch(/attributes\/.+\.rb$/)
  watch(/providers\/.+\.rb$/)
  watch(/recipes\/.+\.rb$/)
  watch(/resources\/.+\.rb$/)
  watch('metadata.rb')
end
```

Note:
Guard is just a filesystem event notifier with plugins
Extreme TDD
Every file save it can trigger actions

---

```
kitchen create REGEX
```

##### Just creates the compute resource

---

```
kitchen converge REGEX
```

1. Creates compute resource (if not created already)
2. Installs Chef (if not created already)
3. Converges cookbooks
4. Leaves compute resource running

---

```
kitchen verify REGEX
```

1. Creates compute resource (if not created already)
2. Installs Chef (if not created already)
3. Converges cookbooks (if not created already)
3. Runs integrations test
  * ServerSpec
  * Rspec
  * BATS
4. Leaves compute resource running

---

```
kitchen test REGEX
```
##### Does ALL THE THINGS

1. Creates compute resource (if not created already)
2. Installs Chef
3. Converges cookbooks
4. Runs integration tests 
5. Destroys compute resource if test pass (default)

---

```
kitchen test -c N REGEX
```
Does <span style="color:#42affa">N</span> number of actions concurrently

---

#### More than just servers

* On the partner engineering team we do many integrations
* Chef is for more than just servers
* Integrations with
  * Citrix Netscalers
  * F5 BigIP
  * VMware vCloud Air
  * EMC CoprHD (ViPR Controller)

---

#### EMC CoprHD (formerly ViPR Controller)
<span style="color:#42affa">(as an example)</span>

* Software defined storage
* Discovers all devices on your network
* Seamless managemnet of heterogenous resources
  * EMC VNX Block
  * ScaleIO
  * Lots [more](http://www.emc.com/techpubs/vipr/what_is_virtual_pool-3.htm)
* Open Source at EMC World 2015 
  * Code available in June
* Great [API Documentation](http://www.emc.com/techpubs/api/vipr/v2-2-0-4/index.htm)

---

#### Fixture Cookbooks
##### Cookbooks to test your (resource) cookbooks 

<span style="color:#42affa">Berksfile</span>
```ruby
source "https://supermarket.chef.io"

metadata

cookbook 'coprhd_host_test', path: 'test/fixtures/cookbooks/coprhd_host_test'
cookbook 'coprhd_vcenter_test', path: 'test/fixtures/cookbooks/coprhd_vcenter_test'
```

---

### Thanks

* [ZenDesk](https://www.zendesk.com/)
* [Little Bean Blue](http://www.littlebeanblue.com.au/)
* [Vibrato](https://hava.io/)
* Franklin Webber (Chef Training Team)
* Joshua Timberman, especially [this post](http://jtimberman.housepub.org/blog/2015/05/15/quick-tip-chefdk-provision/)
* [Soledad Penadés](http://soledadpenades.com/) - Mozilla Evangineer 
* And of course, [Fletcher Nichol](https://github.com/fnichol)
* [Hakim El Hattab](https://github.com/hakimel/reveal.js)
* [Lars Kappert](https://github.com/webpro) for reveal-md

* Deck composed to Okkervil River's [The Stand Ins](https://play.google.com/music/listen?u=0#/album//Okkervil+River/The+Stand+Ins) and [I Am Very Far](https://play.google.com/music/listen?u=0#/album//Okkervil+River/I+Am+Very+Far) albums

---

<svg viewBox="-200 -200 400 400" width="300" height="300" xmlns="http://www.w3.org/2000/svg" class="logo logo-animate">
 <g display="inline" class="ring ring-odd ring-3">
  <title>ring-3</title>
  <path id="svg_6" d="m-159.0006,47.62601c-0.5723,-1.87601 -1.1114,-3.76001 -1.61731,-5.67001c-0.0265,-0.11899 -0.05989,-0.23299 -0.0865,-0.35199c-0.46581,-1.76401 -0.89828,-3.528 -1.3042,-5.311c-0.08,-0.366 -0.1532,-0.726 -0.2397,-1.08501c-0.3394,-1.53099 -0.6588,-3.06799 -0.95169,-4.612c-0.1331,-0.67899 -0.24611,-1.37099 -0.3727,-2.04999c-0.2128,-1.244 -0.4259,-2.46901 -0.6255,-3.70601c-0.1597,-1.052 -0.2995,-2.11699 -0.43932,-3.181c-0.97159,-7.228 -1.49728,-14.56799 -1.49069,-22.022l-33.8606,0c-0.01318,8.11301 0.49918,16.132 1.45073,24.03801l0,0c0.0201,0.166 0.04665,0.339 0.07326,0.51199c0.26628,2.15001 0.55896,4.3 0.89851,6.436c0.09302,0.592 0.19963,1.17799 0.29268,1.763c0.28615,1.757 0.59242,3.50099 0.93176,5.245c0.17303,0.931 0.37941,1.85001 0.56561,2.76801c0.28616,1.39099 0.57906,2.77499 0.8985,4.153c0.27951,1.211 0.57904,2.416 0.88512,3.62c0.27292,1.07201 0.53897,2.15601 0.82515,3.228c0.38612,1.42401 0.79208,2.83501 1.20456,4.246c0.17303,0.599 0.32616,1.198 0.50591,1.77701l0.01991,0c2.3425,7.73898 5.1111,15.28601 8.3253,22.60001l31.0392,-13.64902c-2.6821,-6.06299 -4.9848,-12.32599 -6.92799,-18.74799l0,0z" class="dark" fill="#435363"/>
  <path class="dark" id="svg_7" d="m-0.332,165.44699c-45.70599,0 -87.141,-18.61398 -117.1613,-48.642l-23.9912,23.992c36.13,36.13 86.02249,58.504 141.1525,58.504c101.815,0 185.815,-76.21298 198.10001,-174.70099l-34.18701,0c-12.07199,79.62698 -80.978,140.84698 -163.91299,140.84698z" fill="#F18A20"/>
  <path class="dark" id="svg_8" d="m-0.332,-166.146c37.13499,0 71.44199,12.27849 99.11301,32.9823l20.29199,-27.1924c-33.302,-24.89641 -74.623,-39.64391 -119.39799,-39.64391c-84.718,0 -157.1051,52.77429 -186.09419,127.237l31.5981,12.29201c24.10429,-61.7785 84.25909,-105.675 154.48909,-105.675z" fill="#F18A20"/>
  <path id="svg_9" d="m163.57501,-25.299l34.18597,0c-3.66699,-29.382 -13.71597,-56.78101 -28.70898,-80.7655l-28.78299,17.9615c11.746,18.761 19.858,40.024 23.306,62.804z" fill="#435363"/>
 </g>
 <g display="inline" class="ring ring-even ring-2">
  <title>ring-2</title>
  <path class="dark" id="svg_10" d="m125.06866,-25.29926l34.29993,0c-8.95764,-57.50595 -48.33536,-105.05605 -101.10297,-125.65996l-12.25186,31.49818c40.14304,15.66591 70.4899,51.02397 79.0549,94.16177z" fill="#435363"/>
  <path id="svg_13" d="m-0.33176,127.50653l0,33.77417c80.63214,0 147.66809,-59.35608 159.707,-136.67401l-34.29993,0c-11.64627,58.60403 -63.44888,102.89984 -125.40707,102.89984z" fill="#F18A20"/>
  <path id="svg_12" d="m-128.19452,-0.34961l0,0l-33.77434,0l0,0c0,67.92108 42.12637,126.15909 101.60889,150.03726l12.53804,-31.34509c-47.05094,-18.89362 -80.37259,-64.95958 -80.37259,-118.69217z" fill="#435363"/>
  <path id="svg_11" d="m-0.33176,-128.20572l0,0l0,-33.7742c0,0 0,0 0,0c-68.46011,0 -127.10408,42.79174 -150.62285,103.02628l31.4715,12.24522c18.60076,-47.65655 64.99286,-81.4973 119.15135,-81.4973z" fill="#F18A20"/>
 </g>
 <g display="inline" class="ring ring-odd ring-1">
  <title>ring-1</title>
  <path id="svg_1" d="m-0.33176,89.57959c-49.5865,0 -89.92253,-40.3427 -89.92253,-89.9292c0,-49.5865 40.34268,-89.92253 89.92253,-89.92253c40.92169,0 75.52774,27.4852 86.37543,64.96623l34.79904,0c-11.56641,-56.2881 -61.50562,-98.74708 -121.1678,-98.74708c-68.20724,0 -123.69674,55.48949 -123.69674,123.69672s55.48949,123.7034 123.69674,123.7034c59.66219,0 109.60139,-42.45898 121.1678,-98.7471l-34.79904,0c-10.85434,37.4877 -45.45374,64.97955 -86.37543,64.97955z" fill="#435363"/>
 </g>
 <g display="inline" class="ring ring-even ring-0">
  <title>ring-0</title>
  <path id="svg_2" d="m-60.952,60.26401c15.526,15.53299 36.968,25.14999 60.614,25.14999l0,-35.858c-13.783,0 -26.261,-5.59 -35.299,-14.614l-25.315,25.32201z" fill="#435363"/>
  <path id="svg_3" d="m-86.102,-0.35001c0,12.27901 2.616,23.95201 7.27399,34.52l32.80301,-14.42799c-2.709,-6.14301 -4.21901,-12.931 -4.21901,-20.09201c0,-27.56499 22.341,-49.91199 49.912,-49.91199l0,-35.85101c-47.28999,0 -85.77,38.473 -85.77,85.76299z" class="dark" fill="#F18A20"/>
  <path id="svg_4" d="m30.75999,-80.27l-13.00299,33.42199c10.621,4.13901 19.47899,11.78601 25.12199,21.54201l38.832,0c-7.66599,-25.16901 -26.62599,-45.467 -50.951,-54.964z" fill="#435363"/>
  <path id="svg_5" d="m17.763,46.149l13.004,33.42101c24.311,-9.496 43.27802,-29.79401 50.944,-54.97l-38.832,0c-5.64301,9.763 -14.50099,17.40999 -25.116,21.549z" class="dark" fill="#F18A20"/>
 </g>
</svg>

			    
---
