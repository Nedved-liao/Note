 脚本如下

 
> #!/bin/bash
> 
>  #ENV  
>  DATE=$(date +"%m-%d")  
>  BASE=$(awk -F"'" '/gobin/{print $2}' ~/.bashrc|awk '{print $2}')  
>  FILE=$BASE'log'  
>  LOG=$BASE'nohup.out'
> 
>  >  #Check if the log file exist  
>  if [ ! -d $FILE ];then  
>  mkdir $FILE  
>  fi
> 
>  #Backup the log to the FILE  
>  echo "$DATE"  
>  echo '#Back up the log file#'  
>  cp $LOG $FILE/$DATE.log
> 
>  #Clean today's log  
>  echo 'Start to clear..'  
>  cp /dev/null $LOG
> 
>  #Delete files from seven days ago  
>  find $FILE -type f -mtime +7 -name "*.log" -exec rm {} \;  
>  echo '#Finished.#'
> 
>  
 脚本ENV取当前变量说明

 目的脚本要适应多台服务器中的weblogic域路径,

 利用weblogic用户下的alias(里面在部署项目时候设置有域的bin路径)

 
> cat ~/.bashrc  
>  # .bashrc
> 
>  # Source global definitions  
>  if [ -f /etc/bashrc ]; then  
>  . /etc/bashrc  
>  fi
> 
>  **...**
> 
>  >  export PS1="<\! `hostname` [`whoami`] :"'$PWD'">"  
>  alias gobin='cd /home/weblogic/bea/user_projects/domains/pro_domain/bin/'
> 
>  > **...**
> 
>  
> 
>  # User specific aliases and functions
> 
>  
 截取里面域的路径

 
> BASE=$(awk -F"'" '/gobin/{print $2}' ~/.bashrc|awk '{print $2}')
> 
>  
 

 通过自动获取域的路径,同时通过ansible批发到核心服务当中.

 再配合crontab实现核心服务器日志的定期清理

 

 

  
   
   
 