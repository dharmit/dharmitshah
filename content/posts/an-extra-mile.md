+++
title = "An Extra Mile"                           
date = 2017-12-03T16:19:27+05:30
type = "post"
tags = ["programming"]
+++

Last week, most of my time was occupied in debugging an issue whose cause was
not very obvious when we hit it. A teammate of mine opened a pull request that
failed CI check every single time. He updated the PR about four times but, CI
never felt satisfied. Finally when there was no more code to add to the PR and
CI seemed stubborn, there was a need to dig deeper into it.

A few points (disclaimers, maybe):

- Our project's CI check is not very comprehensive. It's mostly a bunch of
  functional tests that were six months back with then master branch as base
  for tests.

- There are no unit tests.

- CI check prints a lot of logs and does that only when the check fails. When
  everything's OK, it doesn't print more than basic information. When something
  fails assertion checks, it prints logs like crazy:
    - logs from Ansible provisioning
    - logs from all workers in our service
    - nothing in a structured manner

- Tests were written by a single developer with minimal input from others in
  the team. Although that developer did a great job, we didn't add more than a
  few tests or refactor the existing ones.

I had played almost no role in coming up with the tests. So when I had to go
figure why CI checks were failing, I had to learn from scratch about how the
current setup worked.

Our code-base takes good deal of time to get deployed in test environment. We
use Ansible playbook for deploying things in dev, pre-prod and prod
environments. Doing deployment on freshly provisioned nodes would take up to 30
minutes while deployment on existing VMs would take about 15 minutes.

After doing the deployment, I had to go to IPython shell and create a few
dictionaries that were then served to the test suite. These dictionaries mainly
contained IP addresses of the VMs on which tests were to be executed. Finally,
the test suite, created using nosetests, was executed.

Surely, I was seeing similar results in dev environment as CI when the test
suite was executed. Logs made no sense! Assertions failed and it seemed like
they were getting executed before the tasks finished. All I was wondering was,
how are assertions going to succeed when they check result of a task that's
still not finished? And it seemed that way to my teammates as well.

I was about to tell the team that this process is becoming frustrating and not
taking us anywhere when a strange voice in my head asked me to look one more
time - the extra mile!

It took me about 15-20 minutes to find error in one of the worker services.
Surprisingly, the error didn't appear in the CI logs! Nor did it appear in the
log file we generate for every job that passes through the service. It might
have seemed obvious that I should've checked the particular service right from
the beginning. I didn't do that as our team was living in an assumption that
we're spitting out logs for every service in the CI tests; specially when
something fails.

Key lessons that I took away:

- There's immense scope for improvement in our existing CI framework.

- We're doing ourselves and our users a major disservice if we keep focusing on
  adding features and not on fixing a core part like tests.

- Log aggregation needs to improve big time. More importantly, we need to ensure
  that useless actions are not logged!

- Break down the CI check into multiple pieces. One Jenkins job can do the
  deployment, other to run unit tests, yet another to run functional tests, so
  on and so forth. If any one of the jobs fails, we stop the entire thing. That
  helps us save time and, more importantly, figure what piece is exactly
  failing.

- Use less Python magic wherever we can. For example, Ansible playbook can be
  directly executed from shell instead of using Python.

- Focus on this right away instead of waiting for a major issue to bring things
  down!

- Going an extra mile is often rewarding. :wink:

I wrote this blog to bring out my thoughts. In process, if it helps someone
else, I'd be surprised! But if you have any suggestions or comments about how
we can improve, please let me know in comments.

Until next time. :smile:
