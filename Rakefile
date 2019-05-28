task :update do
  system('vagrant box update')
end

task up: :update do
  system('vagrant up')
end

task halt: :up do
  system('vagrant halt')
end

task package: :halt do
  system('vagrant package --base centos7 --output centos7.box')
end

task :clean do
  system('vagrant destroy --force')
  File.delete('centos7.box') if File.exist?('centos7.box')
end
