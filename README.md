Currant blog
=======

Jekyll made blog site

Demo: https://zealous-hill-0a7463803.azurestaticapps.net/

======

### Issue:

Issue 1: Despite of having push command, generate archive workflow does not trigger azure deploy workflow. Follow this solution.

https://github.community/t/push-from-action-does-not-trigger-subsequent-action/16854

Issue 2: If there is any change in post directory, both generate archive and azure deploy workflow will be triggered simultaneously.
When we'll solve the issue #1, after completion of generate archive workflow, azure deploy will get triggered afterwards which at the
end azure deploy workflow will get triggered twice.