node ("windows slave") {

    def config = [:]

    config.rasServer = 'localhost:1545'
    config.racPath = null
    config.v8version = '8.3.10.2699'
    config.infobaseName = 'test1c'
    config.infobaseUser = ''
    config.infobasePwd = ''
    config.deniedMessage = 'Согласованы плановые работы. Установка релиза. Плановое время недоступности с 08:00 по 09:00 МСК 11.09.2019'
    config.permissionCode = '123'

    config.lockInfobase = true
    config.unlockInfobase = true
    config.changeServerParams = false
    config.timeRestart = 10
    
    alternativeRas = new AlternativeRas(this)
    alternativeRas.config << config
    alternativeRas.initialize()
    
    alternativeRas.lockInfobase()
    alternativeRas.changeServerParams()
    alternativeRas.disconnectSessions()
    alternativeRas.unlockInfobase()
    
}


class AlternativeRas {
    
    def context
    def clusterId = null
    def infobaseId = null
    def config = [:]
    
    AlternativeRas(context) {
        
        this.context = context
        
    }
    
    def initialize() {
        
        def pathFile_x86 = "C:/Program\\ Files\\ \\(x86\\)/1cv8/${config.v8version}/bin/rac.exe"
        def pathFile_x64 = "C:/Program\\ Files/1cv8/${config.v8version}/bin/rac.exe"
        
        def possibleLocations = [pathFile_x86, pathFile_x64]
        
        possibleLocations.each { path ->
            def pathResult = this.context.sh(script: "ls $path 2>1 | wc -l", returnStdout: true).trim()
            if (pathResult != "0") {
                config.racPath = path
                return
            }
        }
        
        this.context.print "path rac: ${config.racPath}"
        
        clusterId = this.context.sh(script: "${config.racPath} cluster list ${config.rasServer} | awk 'NR==1{print \$3}'", returnStdout: true).trim()
        this.context.print "cluster id: $clusterId"
        
        infobaseId = this.context.sh(script: "${config.racPath} infobase summary list --cluster=$clusterId ${config.rasServer} | grep -w -B 1 ${config.infobaseName} | sed -n '/infobase/p' | awk '{print \$3}'", returnStdout: true).trim()
        this.context.print "infobase id: $infobaseId"
        
    }

    def changeServerParams() {
    
        this.context.print 'change server params'
        this.context.sh("${config.racPath} cluster update --cluster=$clusterId --lifetime-limit=${config.timeRestart} --expiration-timeout=${config.timeRestart} --kill-problem-processes=yes ${config.rasServer}")
        this.context.sh("sleep 60")
        this.context.print 'return server params'
        this.context.sh("${config.racPath} cluster update --cluster=$clusterId --lifetime-limit=0 --expiration-timeout=0 --kill-problem-processes=no ${config.rasServer}")
       
    }
    
    def disconnectSessions() {
        
        this.context.print "disconnect sessions"
        
        def sessionsStr = this.context.sh(script: "${config.racPath} session list --cluster=$clusterId --infobase=$infobaseId ${config.rasServer} | grep -w '^session\\s' | awk '{print \$3}'", returnStdout: true).trim()
        
        if ( sessionsStr?.trim() ) {
        
            String[] sessions = sessionsStr.split("\\r?\\n")
            this.context.sh("sleep 30")
               
            sessions.each { session ->
                try {
                    this.context.sh("${config.racPath} session terminate --cluster=$clusterId --session=$session ${config.rasServer}")
                    print "terminate session $session"
                } catch (err) {
                    print err
                }
                
            }
        }
        
    }
    
    def lockInfobase() {
        
        this.context.print "lock infobase and lock scheduled-jobs"
        
        this.context.sh("${config.racPath} infobase update --infobase=$infobaseId --infobase-user=${config.infobaseUser} --infobase-pwd=${config.infobasePwd} --cluster=$clusterId --scheduled-jobs-deny=on ${config.rasServer}")
        this.context.sh("${config.racPath} infobase update --infobase=$infobaseId --infobase-user=${config.infobaseUser} --infobase-pwd=${config.infobasePwd} --cluster=$clusterId --sessions-deny=on --denied-message=\"${config.deniedMessage}\" --denied-from=\"\" --permission-code=${config.permissionCode} ${config.rasServer}")
 
    }
    
    def unlockInfobase() {

        this.context.print "unlock infobase and unlock scheduled-jobs"
        this.context.sh("${config.racPath} infobase update --infobase=$infobaseId --infobase-user=${config.infobaseUser} --infobase-pwd=${config.infobasePwd} --cluster=$clusterId --sessions-deny=off --denied-message=\"\" --denied-from=\"\" --permission-code=\"\" ${config.rasServer}")
        this.context.sh("${config.racPath} infobase update --infobase=$infobaseId --infobase-user=${config.infobaseUser} --infobase-pwd=${config.infobasePwd} --cluster=$clusterId --scheduled-jobs-deny=off ${config.rasServer}")
    
    }

}