node ("windows slave") {

    def rasServer = 'localhost:1545'
    def version = '8.3.10.2699'
    def infobaseName = 'test1c'
    def infobaseUser = ''
    def infobasePwd = ''
    def deniedMessage = 'Согласованы плановые работы. Установка релиза. Плановое время недоступности с 08:00 по 09:00 МСК 11.09.2019'
    def permissionCode = '123'

    def lockInfobase = true
    def unlockInfobase = true
    def changeServerParams = false
    def timeRestart = 10
    
    pathFile_x86 = "C:/Program\\ Files\\ \\(x86\\)/1cv8/$version/bin/rac.exe"
    pathFile_x64 = "C:/Program\\ Files/1cv8/$version/bin/rac.exe"
    
    def possibleLocations = [pathFile_x86, pathFile_x64]
    def rac = null
    
    possibleLocations.each { path ->
        def pathResult = sh(script: "ls $path 2>1 | wc -l", returnStdout: true).trim()
        if (pathResult != "0") {
            rac = path
            return
        }
    }
    
    print "path rac: $rac"
    
    def clusterId = sh(script: "$rac cluster list $rasServer | awk 'NR==1{print \$3}'", returnStdout: true).trim()
    
    print "cluster id: $clusterId"
    
    def infobaseId = sh(script: "$rac infobase summary list --cluster=$clusterId $rasServer | grep -w -B 1 $infobaseName | sed -n '/infobase/p' | awk '{print \$3}'", returnStdout: true).trim()
    
    print "infobase id: $infobaseId"
    
    if (lockInfobase) {
       
       print 'begin lock infobase and disconnect user'
       
       if (changeServerParams) {
            print 'change server params'
            sh("$rac cluster update --cluster=$clusterId --lifetime-limit=$timeRestart --expiration-timeout=$timeRestart --kill-problem-processes=yes $rasServer")
            sh("sleep 60")
            print 'return server params'
            sh("$rac cluster update --cluster=$clusterId --lifetime-limit=0 --expiration-timeout=0 --kill-problem-processes=no $rasServer")
       }
       
       print "lock infobase and lock scheduled-jobs"
       
       sh("$rac infobase update --infobase=$infobaseId --infobase-user=$infobaseUser --infobase-pwd=$infobasePwd --cluster=$clusterId --scheduled-jobs-deny=on $rasServer")
       sh("$rac infobase update --infobase=$infobaseId --infobase-user=$infobaseUser --infobase-pwd=$infobasePwd --cluster=$clusterId --sessions-deny=on --denied-message=\"$deniedMessage\" --denied-from=\"\" --permission-code=$permissionCode $rasServer")
       
       def sessionsStr = sh(script: "$rac session list --cluster=$clusterId --infobase=$infobaseId $rasServer | grep -w '^session\\s' | awk '{print \$3}'", returnStdout: true).trim()
       String[] sessions = sessionsStr.split("\\r?\\n")
       
       sleep 30
       
       sessions.each { session ->
            try {
                sh("$rac session terminate --cluster=$clusterId --session=$session $rasServer")
                print "terminate session $session"
            } catch (err) {
                print err
            }
            
       }
       
    }
    
    if (unlockInfobase) {
        
        print "unlock infobase and unlock scheduled-jobs"
        sh("$rac infobase update --infobase=$infobaseId --infobase-user=$infobaseUser --infobase-pwd=$infobasePwd --cluster=$clusterId --sessions-deny=off --denied-message=\"\" --denied-from=\"\" --permission-code=\"\" $rasServer")
        sh("$rac infobase update --infobase=$infobaseId --infobase-user=$infobaseUser --infobase-pwd=$infobasePwd --cluster=$clusterId --scheduled-jobs-deny=off $rasServer")   
    }
  
}