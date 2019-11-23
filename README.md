# Servidor DNS

![DNS](image/DNS.png)

### En este bloque vamos a estudiar cómo funciona el protocolo DNS, y vamos a configurar distisntos servidores DNS. El DNS se utiliza para distintos propósitos. Los más comunes son:

* Resolución de nombres: Dado el nombre completo de un host obtener su dirección IP.
* Resolución inversa de direcciones: Es el mecanismo inverso al anterior. Consiste en, dada una dirección IP, obtener el nombre asociado a la misma.
* Resolución de servidores de correo: Dado un nombre de dominio, obtener el servidor a través del cual debe realizarse la entrega del correo electrónico.

### Los contenidos que vamos a estudiar en este bloque serán:

* Cómo funciona el protocolo DNS
* Realizar consultas a los servidores DNS
* Estudio de distintos servidores DNS: DNS Windows Server, dnsmasq, bind9
* Servidores DNS esclavos
* Configuración de subdominios, delegación de subdominios.
* Servidores DNS dinámicos

## Práctica

### Escenario

Vamos a trabajar en esta práctica con varias máquinas virtuales en vagrant y en ellas crearemos el siguiente escenario:

1. En nuestra red local tenemos un servidor Web que sirve dos páginas web: www.iesgn.org, departamentos.iesgn.org
2. Vamos a instalar en nuestra red local un servidor DNS (lo puedes instalar en el mismo equipo que tiene el servidor web)
3. Voy a suponer en este documento que el nombre del servidor DNS va a ser vagrant.iesgn.org. El nombre del servidor de tus prácticas será tunombre.iesgn.org.

El Vagrantfile que utilizaremos es:

~~~
Vagrant.configure("2") do |config|

  config.vm.define :servidor1 do |servidor1|
    servidor1.vm.box = "debian/buster64"
    servidor1.vm.hostname = "ServidorMaster"
    servidor1.vm.network :public_network,:bridge=>"wlp2s0"
    #servidor1.vm.network :public_network,:bridge=>"enp3s0"
    servidor1.vm.network :private_network, ip: "10.0.0.1", virtualbox__intnet: "miredinterna"
  end

  config.vm.define :servidor2 do |servidor2|
    servidor2.vm.box = "debian/buster64"
    servidor2.vm.hostname = "ServidorSlave"
    servidor2.vm.network :public_network,:bridge=>"wlp2s0"
    #servidor2.vm.network :public_network,:bridge=>"enp3s0"
    servidor2.vm.network :private_network, ip: "10.0.0.2", virtualbox__intnet: "miredinterna"
  end

   config.vm.define :servidor3 do |servidor3|
    servidor3.vm.box = "debian/buster64"
    servidor3.vm.hostname = "ServidorSubDominio"
    servidor3.vm.network :public_network,:bridge=>"wlp2s0"
    #servidor3.vm.network :public_network,:bridge=>"enp3s0"
    servidor3.vm.network :private_network, ip: "10.0.0.3", virtualbox__intnet: "miredinterna"
  end

  config.vm.define :cliente1 do |cliente1|
     cliente1.vm.box = "debian/buster64"
    cliente1.vm.hostname = "Cliente1" 
    cliente1.vm.network :public_network,:bridge=>"wlp2s0"
    cliente1.vm.network :private_network, ip: "10.0.0.4", virtualbox__intnet: "miredinterna"
  end

end
~~~
