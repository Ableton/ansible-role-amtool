library(identifier: 'ableton-utils@nre/master/4646-ansible-utils', changelog: false)
library(identifier: 'groovylint@0.13', changelog: false)
library(identifier: 'python-utils@0.13', changelog: false)


devToolsProject.run(
  setup: { data ->
    Object venv = pyenv.createVirtualEnv(readFile('.python-version'))
    venv.run('pip install -r requirements-dev.txt')
    venv.inside {
      ansibleUtils.galaxyInstall()
    }
    data['venv'] = venv
  },
  test: { data ->
    data.venv.inside {
      parallel(failFast: false,
        'ansible-lint': {
          String stdout = sh(
            label: 'ansible-lint',
            returnStdout: true,
            script: 'ansible-lint --offline -c .ansible-lint.yml',
          )

          // If only warnings are found, ansible-lint will exit with code 0 but still
          // write an error summary to stdout. It's not possible to treat warnings as
          // errors, and likely will never be. See:
          // https://github.com/ansible-community/ansible-lint/issues/236
          if (stdout) {
            error 'ansible-lint exited with warnings, check output of the previous step'
          }
        },
        // groovylint: { groovylint.checkSingleFile(path: './Jenkinsfile') },
        molecule: { ansibleUtils.molecule() },
        // yamllint: { sh 'yamllint --strict .' },
      )
    }
  },
  deployWhen: { devToolsProject.shouldDeploy(defaultBranch: 'main') },
  deploy: { data ->
    venv.inside { ansibleUtils.publishRole('ansible-galaxy-api-key') }
  },
)
