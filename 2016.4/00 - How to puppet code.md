## Some how-tos in Puppet

_Note : You need to have some Puppet knowledge to understand some things in this page._


### Disable puppet agent run in all nodes of an environment

Put the below code in one of the global manifests of your environment.

 ```
 /etc/puppetlabs/code/environments/production/manifests/site.pp
 
 service { 'puppet' :
  ensure => stopped,
  enable => false,
 }
 ```

### Create a file but take a backup if the file already exists ( and the content is not same )

```
file { '/tmp/testfile.txt' :
   ensure => present,
   backup => true,
   content => "hello world",
}

# Creates a testfile.txt.puppet-bak file in the same directory ( /tmp ) if the file exists and the content is different.
```

You can have you own extension by putting that as a value to the backup attribute above. For eg.

```
file { '/tmp/testfile.txt' :
   ensure => present,
   backup => '.bkp',
   content => "hello world",
}

# Same function as above but the backup file name is "testfile.txt.bkp"
```

_Imp Note : Puppet doesn't create mutiple backup files. So old backup file will be replaced with new backup. 
You may want to look at filebucket [here](https://docs.puppet.com/puppet/4.7/types/file.html#file-attribute-backup) to have more control. But it is slightly more complicated to use._

### Copy whole directory content

```
file { 'copy directory' :
  ensure => directory,
  path => '/tmp/copied_dir',
  source => 'puppet:///modules/filecopy/folder_to_be_copied/',   
  # or source => '/NFS_mount_path_to_folder/folder_to_be_copied' 
  recurse => true,
}
```
