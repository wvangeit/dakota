.. _ngw-node-dispatchAndCollect:

==================
dispatchAndCollect
==================

-----------
Description
-----------

The dispatchAndCollect node is used to submit a workflow to a remote machine. It is designed to ingest a Dakota
parameter file and produce results that will be written to a Dakota results file. Because it takes a single
set of Dakota parameters and produces a single set of Dakota results, this node is
**intended to be run as part of a Dakota analysis driver.** That is, a dispatchAndCollect node is not designed to iterate
over a parameter space, but is itself driven by Dakota.

The dispatchAndCollect node's name comes from its two-step strategy. It is intended to be run asynchronously
with Dakota (in what is sometimes called "offline mode"). After this node submits a workflow to a remote machine (i.e. the dispatch step),
the node will exit early, returning fail values to Dakota. Even though they are "fail" values, this immediate return
indicates that the jobs are off running somewhere else. Then, at a later date, the analyst is expected to run Dakota
at least one more time. When it's executed again, the dispatchAndCollect node is smart enough to pick up the data
generated by each job on the remote machine and return it to Dakota (i.e. the collect step), rather than send
fresh values for remote job submission.

In short - it is not necessary to leave Dakota up and running while you wait for all your remote job evaluations to complete.

-----
Notes
-----

- To see the dispatchAndCollect node in action, take a look at :ref:`the official NGW job submision examples <gui-job-submission-workflow>`.
- There is a variaton of the dispatchAndCollect node that is intended to enable asynchronous, "offline mode"
  runs of Dakota on your local machine, relying on local process management instead of remote job submission.
  See :ref:`localDispatchAndCollect <ngw-node-localDispatchAndCollect>`.

----------
Properties
----------

**Dispatcher Workflow Settings**

- **dispatchedWorkflow:** The path to the "inner" workflow that this node will dispatch to the remote machine.
- **failValue:** The value used for the dispatchAndCollect node to recognize that the evaluation failed for whatever
  reason on the remote machine. "Fail" is used somewhat broadly here, as any non-success job status is considered a failure
  (all possible job statuses are enumerated below). Typically, "NaN" is used to designate a fail value, but you are allowed to change it
  if "NaN" already has reserved meaning according to your driver. However, you must select *some* reserved value to indicate failure.
- **dispatcherInExpertMode:** Use this if you want the dispatched workflow to run in :ref:`expert mode.<ngw-expertmode>`
  For most simple examples, this can be set to false.
- **rerunFailedEvaluations:** Set this to true to force the evaluation to re-run - even if data already exists in the
  evaluation directory - but only if the previous evaluation failed.

**Remote Submission**

- **account** - The WCID number to use for job submission. Talk to your system administrator to request an account WCID number.
- **job.hours** - The number of hours of queue time to request.
- **job.minutes** - The number of minutes of queue time to request.
- **num.nodes** - The number of compute nodes to request.
- **num.processors** - The total number of processors to request.
- **queue** - The partition (queue name) to pass to Slurm - by default, 'batch', but 'short' is an option too.

**Script Substitution**

- **submitScript** - Use this field if you would like to supply your own script responsible for submitting to the job queue, replacing the ``submit-dispatch.sh``
  provided by default.
- **statusScript** - Use this field if you would like to supply your own script responsible for submitting to the job queue, replacing the ``status.sh``
  provided by default.
- **checkjobScript** - Use this field if you would like to supply your own script responsible for submitting to the job queue, replacing the ``checkjob.sh``
  provided by default.
- **cancelScript** - Use this field if you would like to supply your own script responsible for submitting to the job queue, replacing the ``cancel.sh``
  provided by default.
- **dispatchWorkflowScript** - Use this field if you would like to supply your own script responsible for running the dispatched workflow, replacing the ``dispatchWorkflowRemote.sh``
  provided by default.

-----------
Input Ports
-----------

- **paramsFile:** The Dakota parameters file which provides the input parameters to the workflow that will be dispatched
  to the remote machine.

------------
Output Ports
------------

- **responsesMap:** A map of response labels and response values. This map can be passed to a :ref:`dakotaResultsFile <ngw-node-dakotaResultsFile>` node for further processing.
- **jobStatus:** The job status from the remote machine. This is a number value, enumerated below.

-----------------------
Interpreting Job Status
-----------------------

The remote machine will only return a number value to indicate job status. Here is what each number means:

- 0: SUCCESSFUL
- 1: COMPLETED
- 2: BOOT_FAIL
- 3: DEADLINE
- 4: FAILED
- 5: NODE_FAIL
- 6: OUT_OF_MEMORY
- 7: PREEMPTED
- 8: TIMEOUT
- 9: CANCELLED
- 10: UNDEFINED

-------------------------------------
Usage Notes - Editing Control Scripts
-------------------------------------

Refer to the main documentation for each control script to learn more about its function and what to consider editing:

- :ref:`submit-dispatch.sh <gui-job-submission-workflow-dispatchAndCollect-scripts-submitDispatch>`
- :ref:`status.sh <gui-job-submission-workflow-common-scripts-status>`
- :ref:`checkjob.sh <gui-job-submission-workflow-common-scripts-checkjob>`
- :ref:`cancel.sh <gui-job-submission-workflow-dakotaQueueSubmit-scripts-cancel>`
- :ref:`dispatchWorkflowRemote.sh <gui-job-submission-workflow-dispatchAndCollect-scripts-dispatchWorkflowRemote>`
