task :up do
  system('vagrant up')
end

task halt: :up do
  system('vagrant halt')
end

task package: :halt do
  system('vagrant package --base centos7 --output centos7.box')
end
