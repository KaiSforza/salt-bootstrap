pipeline {
    agent { label 'bootstrap' }
    stages {
        stage('shellcheck') {
            steps {
                sh 'shellcheck -s sh -f checkstyle bootstrap-salt.sh | tee checkstyle.xml'
                checkstyle pattern: '**/checkstyle.xml'
                archiveArtifacts artifacts: '**/checkstyle.xml'
            }
        }
        stage('kitchen') {
            script {
                import java.util.Random

                Random rand = new Random()

                // ONLY CHANGE THIS BIT PLEASE
                def baseDistros = ["debian8",
                                   "debian9",
                                   "suse",
                                   "centos6",
                                   "arch",
                                   "ubuntu-14.04",
                                   "ubuntu-18.04",
                                   "windows",
                                   ]
                def versions = ["stable", "stable-old", "git"]

                def basePrDistros = ["ubuntu-16.04",
                                     "centos7"]

                def prVersions = ["stable", "git"]

                // You probably shouldn't be messing with this stuff down here
                // Combines lists (Because we can't use .combinations())

                def listCombinations(a, b) {
                    def ret = []
                    for (i in a) {
                        for (j in b) {
                            ret = ret + [[i, j]]
                        }
                    }
                    return ret
                }

                def distros = (baseDistros + basePrDistros).unique()

                // Runs actual kitchen scripts for the distros and versions
                def runKitchen(String distro, String version) {
                    def runon = "${distro}-${version}"
                    stage ("kitchen-${runon}") {
                        echo "kitchen create ${runon}"
                        echo "kitchen converge ${runon}"
                        echo "kitchen destroy ${runon}"
                    }
                }

                def distroversions = listCombinations(distros, versions)

                def prDistros = (basePrDistros + distros[rand.nextInt(baseDistros.size())]).unique()

                def prDistroversions = listCombinations(prDistros, prVersions)

                // Creates the nodes for each runKitchen function
                def makeSetupRuns(d, v) {
                    return {
                        node {
                            runKitchen(d, v)
                            checkpoint "kitchen-${d}-${v}"
                        }
                    }
                }

                def setupRuns = distroversions.collectEntries {
                    ["kitchen-${it[0]}-${it[1]}" : makeSetupRuns(it[0], it[1])]
                }

                def prSetupRuns = prDistroversions.collectEntries {
                    ["kitchen-${it[0]}-${it[1]}" : makeSetupRuns(it[0], it[1])]
                }
                if (env.CHANGE_ID) {
                    // Running for a PR only runs against 4 random distros from a shorter list
                    stage('kitchen-pr') {
                        parallel prSetupRuns
                    }
                } else {
                    // If we're not running for a pr we run *everything*
                    stage('kitchen-all') {
                        parallel setupRuns
                    }
                }
            }
        }
    }
}


/*
 * TODO:
 * 1. Tests for each supported distro in bootstrap + branch shellcheck test (Shellcheck should be done)
 * 2. Each distro needs a "stable" install (installs stable packages from our repo) and a "git" install (installs off of a git tag)
 * 3. Running against each branch (stable, develop)
 * 4. And probably a small subset against each pull request (similar to what we do in salt)
 *
 * Distros to check:
 *     Debian 8
 *     Suse 42.1
 *     CentOS 7
 *     CentOS 6
 *     Arch
 *     Ubuntu 16.04
 *     Ubuntu 14.04
 *     Windows
 *
 * Runs each against develop and stable
 *
 */
