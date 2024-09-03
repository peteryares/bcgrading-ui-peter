<script>
    import { onMount } from 'svelte';
    import { page } from '$app/stores';

    let classDetails = null;
    let gradelistEntries = [];
    let error = '';

    const { classid } = $page.params;

    onMount(async () => {
        try {
            const response = await fetch(`http://localhost:4000/registrar/${classid}`, {
                headers: {
                    'Authorization': `Bearer ${localStorage.getItem('jwtToken')}`
                }
            });

            if (response.ok) {
                const data = await response.json();
                classDetails = data.classDetails;
                gradelistEntries = data.gradelistEntries;
            } else {
                error = `Failed to fetch grades: ${response.statusText}`;
            }
        } catch (err) {
            error = `Error fetching grades: ${err.message}`;
        }
    });

    function filterGradesByTerm(term) {
        return gradelistEntries.filter(entry => entry.term === term);
    }
</script>

<div class="container mt-4">
    <h2>Grades for Class ID: {classid}</h2>

    {#if error}
        <div class="alert alert-danger">{error}</div>
    {/if}

    {#if classDetails && gradelistEntries.length > 0}
        <!-- <div class="class-details">
            <h3>Class Details</h3>
            <p>Subject Code: {classDetails.subjectcode}</p>
            <p>Semester: {classDetails.semester}</p>
            <p>Year: {classDetails.year}</p>
            <p>Teacher ID: {classDetails.teacherid}</p>
            <p>Status: {classDetails.isActive ? 'Active' : 'Inactive'}</p>
        </div> -->

        <h3>Grade List</h3>

        <!-- Tabs for each Term -->
        <ul class="nav nav-tabs" id="termTab" role="tablist">
            <li class="nav-item" role="presentation">
                <button class="nav-link active" id="prelim-tab" data-bs-toggle="tab" data-bs-target="#prelim" type="button" role="tab" aria-controls="prelim" aria-selected="true">Prelim</button>
            </li>
            <li class="nav-item" role="presentation">
                <button class="nav-link" id="midterm-tab" data-bs-toggle="tab" data-bs-target="#midterm" type="button" role="tab" aria-controls="midterm" aria-selected="false">Midterm</button>
            </li>
            <li class="nav-item" role="presentation">
                <button class="nav-link" id="final-tab" data-bs-toggle="tab" data-bs-target="#final" type="button" role="tab" aria-controls="final" aria-selected="false">Final</button>
            </li>
        </ul>

        <!-- Term Content -->
        <div class="tab-content" id="termTabContent">
            <!-- Prelim Term -->
            <div class="tab-pane fade show active" id="prelim" role="tabpanel" aria-labelledby="prelim-tab">
                {#if filterGradesByTerm('Prelim').length > 0}
                    <!-- Sub-tabs for Prelim term -->
                    <ul class="nav nav-tabs mt-3" id="prelimSubTab" role="tablist">
                        <li class="nav-item" role="presentation">
                            <button class="nav-link active" id="prelim-attendance-tab" data-bs-toggle="tab" data-bs-target="#prelim-attendance" type="button" role="tab" aria-controls="prelim-attendance" aria-selected="true">Attendance</button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" id="prelim-participation-tab" data-bs-toggle="tab" data-bs-target="#prelim-participation" type="button" role="tab" aria-controls="prelim-participation" aria-selected="false">Participation</button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" id="prelim-quiz-tab" data-bs-toggle="tab" data-bs-target="#prelim-quiz" type="button" role="tab" aria-controls="prelim-quiz" aria-selected="false">Quiz</button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" id="prelim-activityproject-tab" data-bs-toggle="tab" data-bs-target="#prelim-activityproject" type="button" role="tab" aria-controls="prelim-activityproject" aria-selected="false">Activity Project</button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" id="prelim-exam-tab" data-bs-toggle="tab" data-bs-target="#prelim-exam" type="button" role="tab" aria-controls="prelim-exam" aria-selected="false">Exam</button>
                        </li>
                    </ul>

                    <!-- Sub-tab content for Prelim -->
                    <div class="tab-content mt-3" id="prelimSubTabContent">
                        <div class="tab-pane fade show active" id="prelim-attendance" role="tabpanel" aria-labelledby="prelim-attendance-tab">
                            {#each filterGradesByTerm('Prelim') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Attendance Counter: {entry.AttendanceScore.attendancecounter}</p>
                                <p>Attendance Scores: 
                                    {#each Array.from({length: entry.AttendanceScore.attendancecounter}, (_, i) => i + 1) as i}
                                        {#if entry.AttendanceScore['attendance_' + i] !== undefined}
                                            Attendance {i}: {entry.AttendanceScore['attendance_' + i]} &nbsp;
                                        {/if}
                                    {/each}
                                </p>
                                <p>Total: {entry.AttendanceScore.attendance_total}</p>
                                <hr>
                            {/each}
                        </div>
                        <div class="tab-pane fade" id="prelim-participation" role="tabpanel" aria-labelledby="prelim-participation-tab">
                            {#each filterGradesByTerm('Prelim') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Participation Scores: 
                                    {#each Array.from({length: 10}, (_, i) => i + 1) as i}
                                        {#if entry.ParticipationScore['participation_' + i] !== undefined}
                                            Participation {i}: {entry.ParticipationScore['participation_' + i]} &nbsp;
                                        {/if}
                                    {/each}
                                </p>
                                <p>Total: {entry.ParticipationScore.participation_total}</p>
                                <hr>
                            {/each}
                        </div>
                        <div class="tab-pane fade" id="prelim-quiz" role="tabpanel" aria-labelledby="prelim-quiz-tab">
                            {#each filterGradesByTerm('Prelim') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Quiz Scores: 
                                    {#each Array.from({length: 10}, (_, i) => i + 1) as i}
                                        {#if entry.QuizScore['quiz_' + i] !== undefined}
                                            Quiz {i}: {entry.QuizScore['quiz_' + i]} &nbsp;
                                        {/if}
                                    {/each}
                                </p>
                                <p>Total: {entry.QuizScore.quiz_total}</p>
                                <hr>
                            {/each}
                        </div>
                        <div class="tab-pane fade" id="prelim-activityproject" role="tabpanel" aria-labelledby="prelim-activityproject-tab">
                            {#each filterGradesByTerm('Prelim') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Activity Project Scores: 
                                    {#each Array.from({length: 10}, (_, i) => i + 1) as i}
                                        {#if entry.ActivityProjectScore['activityproject_' + i] !== undefined}
                                            Activity Project {i}: {entry.ActivityProjectScore['activityproject_' + i]} &nbsp;
                                        {/if}
                                    {/each}
                                </p>
                                <p>Total: {entry.ActivityProjectScore.activityproject_total}</p>
                                <hr>
                            {/each}
                        </div>
                        <div class="tab-pane fade" id="prelim-exam" role="tabpanel" aria-labelledby="prelim-exam-tab">
                            {#each filterGradesByTerm('Prelim') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Exam Score: Exam 1: {entry.ExamScore.exam_1}</p>
                                <p>Total: {entry.ExamScore.exam_total}</p>
                                <hr>
                            {/each}
                        </div>
                    </div>
                {:else}
                    <p>No data available for Prelim term.</p>
                {/if}
            </div>

            <!-- Midterm Term -->
            <div class="tab-pane fade" id="midterm" role="tabpanel" aria-labelledby="midterm-tab">
                {#if filterGradesByTerm('Midterm').length > 0}
                    <!-- Sub-tabs for Midterm term -->
                    <ul class="nav nav-tabs mt-3" id="midtermSubTab" role="tablist">
                        <li class="nav-item" role="presentation">
                            <button class="nav-link active" id="midterm-attendance-tab" data-bs-toggle="tab" data-bs-target="#midterm-attendance" type="button" role="tab" aria-controls="midterm-attendance" aria-selected="true">Attendance</button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" id="midterm-participation-tab" data-bs-toggle="tab" data-bs-target="#midterm-participation" type="button" role="tab" aria-controls="midterm-participation" aria-selected="false">Participation</button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" id="midterm-quiz-tab" data-bs-toggle="tab" data-bs-target="#midterm-quiz" type="button" role="tab" aria-controls="midterm-quiz" aria-selected="false">Quiz</button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" id="midterm-activityproject-tab" data-bs-toggle="tab" data-bs-target="#midterm-activityproject" type="button" role="tab" aria-controls="midterm-activityproject" aria-selected="false">Activity Project</button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" id="midterm-exam-tab" data-bs-toggle="tab" data-bs-target="#midterm-exam" type="button" role="tab" aria-controls="midterm-exam" aria-selected="false">Exam</button>
                        </li>
                    </ul>

                    <!-- Sub-tab content for Midterm -->
                    <div class="tab-content mt-3" id="midtermSubTabContent">
                        <div class="tab-pane fade show active" id="midterm-attendance" role="tabpanel" aria-labelledby="midterm-attendance-tab">
                            {#each filterGradesByTerm('Midterm') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Attendance Counter: {entry.AttendanceScore.attendancecounter}</p>
                                <p>Attendance Scores: 
                                    {#each Array.from({length: entry.AttendanceScore.attendancecounter}, (_, i) => i + 1) as i}
                                        {#if entry.AttendanceScore['attendance_' + i] !== undefined}
                                            Attendance {i}: {entry.AttendanceScore['attendance_' + i]} &nbsp;
                                        {/if}
                                    {/each}
                                </p>
                                <p>Total: {entry.AttendanceScore.attendance_total}</p>
                                <hr>
                            {/each}
                        </div>
                        <div class="tab-pane fade" id="midterm-participation" role="tabpanel" aria-labelledby="midterm-participation-tab">
                            {#each filterGradesByTerm('Midterm') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Participation Scores: 
                                    {#each Array.from({length: 10}, (_, i) => i + 1) as i}
                                        {#if entry.ParticipationScore['participation_' + i] !== undefined}
                                            Participation {i}: {entry.ParticipationScore['participation_' + i]} &nbsp;
                                        {/if}
                                    {/each}
                                </p>
                                <p>Total: {entry.ParticipationScore.participation_total}</p>
                                <hr>
                            {/each}
                        </div>
                        <div class="tab-pane fade" id="midterm-quiz" role="tabpanel" aria-labelledby="midterm-quiz-tab">
                            {#each filterGradesByTerm('Midterm') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Quiz Scores: 
                                    {#each Array.from({length: 10}, (_, i) => i + 1) as i}
                                        {#if entry.QuizScore['quiz_' + i] !== undefined}
                                            Quiz {i}: {entry.QuizScore['quiz_' + i]} &nbsp;
                                        {/if}
                                    {/each}
                                </p>
                                <p>Total: {entry.QuizScore.quiz_total}</p>
                                <hr>
                            {/each}
                        </div>
                        <div class="tab-pane fade" id="midterm-activityproject" role="tabpanel" aria-labelledby="midterm-activityproject-tab">
                            {#each filterGradesByTerm('Midterm') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Activity Project Scores: 
                                    {#each Array.from({length: 10}, (_, i) => i + 1) as i}
                                        {#if entry.ActivityProjectScore['activityproject_' + i] !== undefined}
                                            Activity Project {i}: {entry.ActivityProjectScore['activityproject_' + i]} &nbsp;
                                        {/if}
                                    {/each}
                                </p>
                                <p>Total: {entry.ActivityProjectScore.activityproject_total}</p>
                                <hr>
                            {/each}
                        </div>
                        <div class="tab-pane fade" id="midterm-exam" role="tabpanel" aria-labelledby="midterm-exam-tab">
                            {#each filterGradesByTerm('Midterm') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Exam Score: Exam 1: {entry.ExamScore.exam_1}</p>
                                <p>Total: {entry.ExamScore.exam_total}</p>
                                <hr>
                            {/each}
                        </div>
                    </div>
                {:else}
                    <p>No data available for Midterm term.</p>
                {/if}
            </div>

            <!-- Final Term -->
            <div class="tab-pane fade" id="final" role="tabpanel" aria-labelledby="final-tab">
                {#if filterGradesByTerm('Finals').length > 0}
                    <!-- Sub-tabs for Final term -->
                    <ul class="nav nav-tabs mt-3" id="finalSubTab" role="tablist">
                        <li class="nav-item" role="presentation">
                            <button class="nav-link active" id="final-attendance-tab" data-bs-toggle="tab" data-bs-target="#final-attendance" type="button" role="tab" aria-controls="final-attendance" aria-selected="true">Attendance</button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" id="final-participation-tab" data-bs-toggle="tab" data-bs-target="#final-participation" type="button" role="tab" aria-controls="final-participation" aria-selected="false">Participation</button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" id="final-quiz-tab" data-bs-toggle="tab" data-bs-target="#final-quiz" type="button" role="tab" aria-controls="final-quiz" aria-selected="false">Quiz</button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" id="final-activityproject-tab" data-bs-toggle="tab" data-bs-target="#final-activityproject" type="button" role="tab" aria-controls="final-activityproject" aria-selected="false">Activity Project</button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" id="final-exam-tab" data-bs-toggle="tab" data-bs-target="#final-exam" type="button" role="tab" aria-controls="final-exam" aria-selected="false">Exam</button>
                        </li>
                    </ul>

                    <!-- Sub-tab content for Final -->
                    <div class="tab-content mt-3" id="finalSubTabContent">
                        <div class="tab-pane fade show active" id="final-attendance" role="tabpanel" aria-labelledby="final-attendance-tab">
                            {#each filterGradesByTerm('Finals') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Attendance Counter: {entry.AttendanceScore.attendancecounter}</p>
                                <p>Attendance Scores: 
                                    {#each Array.from({length: entry.AttendanceScore.attendancecounter}, (_, i) => i + 1) as i}
                                        {#if entry.AttendanceScore['attendance_' + i] !== undefined}
                                            Attendance {i}: {entry.AttendanceScore['attendance_' + i]} &nbsp;
                                        {/if}
                                    {/each}
                                </p>
                                <p>Total: {entry.AttendanceScore.attendance_total}</p>
                                <hr>
                            {/each}
                        </div>
                        <div class="tab-pane fade" id="final-participation" role="tabpanel" aria-labelledby="final-participation-tab">
                            {#each filterGradesByTerm('Finals') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Participation Scores: 
                                    {#each Array.from({length: 10}, (_, i) => i + 1) as i}
                                        {#if entry.ParticipationScore['participation_' + i] !== undefined}
                                            Participation {i}: {entry.ParticipationScore['participation_' + i]} &nbsp;
                                        {/if}
                                    {/each}
                                </p>
                                <p>Total: {entry.ParticipationScore.participation_total}</p>
                                <hr>
                            {/each}
                        </div>
                        <div class="tab-pane fade" id="final-quiz" role="tabpanel" aria-labelledby="final-quiz-tab">
                            {#each filterGradesByTerm('Finals') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Quiz Scores: 
                                    {#each Array.from({length: 10}, (_, i) => i + 1) as i}
                                        {#if entry.QuizScore['quiz_' + i] !== undefined}
                                            Quiz {i}: {entry.QuizScore['quiz_' + i]} &nbsp;
                                        {/if}
                                    {/each}
                                </p>
                                <p>Total: {entry.QuizScore.quiz_total}</p>
                                <hr>
                            {/each}
                        </div>
                        <div class="tab-pane fade" id="final-activityproject" role="tabpanel" aria-labelledby="final-activityproject-tab">
                            {#each filterGradesByTerm('Finals') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Activity Project Scores: 
                                    {#each Array.from({length: 10}, (_, i) => i + 1) as i}
                                        {#if entry.ActivityProjectScore['activityproject_' + i] !== undefined}
                                            Activity Project {i}: {entry.ActivityProjectScore['activityproject_' + i]} &nbsp;
                                        {/if}
                                    {/each}
                                </p>
                                <p>Total: {entry.ActivityProjectScore.activityproject_total}</p>
                                <hr>
                            {/each}
                        </div>
                        <div class="tab-pane fade" id="final-exam" role="tabpanel" aria-labelledby="final-exam-tab">
                            {#each filterGradesByTerm('Finals') as entry}
                                <p>{entry.studentFirstName} {entry.studentLastName}</p>
                                <p>Exam Score: Exam 1: {entry.ExamScore.exam_1}</p>
                                <p>Total: {entry.ExamScore.exam_total}</p>
                                <hr>
                            {/each}
                        </div>
                    </div>
                {:else}
                    <p>No data available for Final term.</p>
                {/if}
            </div>
        </div>
    {:else}
        <p>No grades available for this class.</p>
    {/if}
</div>

<style>
    .container {
        max-width: 900px;
        margin-top: 20px;
    }
    .class-details {
        margin-bottom: 20px;
    }
    .nav-tabs {
        margin-bottom: 20px;
    }
</style>
