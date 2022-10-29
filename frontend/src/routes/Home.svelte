<script>
    import fastapi from "../lib/api"
    import { link } from 'svelte-spa-router'
    import { page } from "../lib/store"
    import moment from 'moment/min/moment-with-locales'
    moment.locale('ko')

    let question_list = []
    let size = 10
    let total = 0
    $: total_page = Math.ceil(total/size)
  
    function get_question_list(_page) {
        let params = {
            page: _page,
            size: size,
        }
        fastapi('get', '/api/question/list', params, (json) => {
            question_list = json.question_list
            $page = _page
            total = json.total
        })
    }

    $: get_question_list($page)
</script>

<div class="container my-3">
    <table class="table">
        <thead>
            <tr class="table-dark">
                <th>번호</th>
                <th>제목</th>
                <th>작성일시</th>
            </tr>
        </thead>
        <tbody>
            {#each question_list as question, i}
            <tr>
                <td>{i+1}</td>
                <td>
                    <a use:link href="/detail/{question.id}">{question.subject}</a>
                </td>
                <td>{moment(question.create_date).format("YYYY년 MM월 DD일 hh:mm a")}</td>
            </tr>
            {/each}
        </tbody>
    </table>
    <!-- 페이징 처리 시작 -->
    <ul class="pagination justify-content-center">
        <!-- 이전 페이지 -->
        <li class="page-item {$page <= 0 && 'disabled'}">
            <button class="page-link" on:click="{() => get_question_list($page-1)}">이전</button>
        </li>
        <!-- 페이지 번호 -->
        {#each Array(total_page) as _, loop_page}
        <!-- 첫번째 페이지는 강제로 표시 -->
        {#if loop_page === 0}
            <li class="page-item {loop_page === $page && 'active'}">
                <button class="page-link" on:click="{() => get_question_list(loop_page)}">{loop_page+1}</button>
            </li>
        <!-- 마지막 페이지도 강제로 표시 -->
        {:else if loop_page === total_page-1}
            <li class="page-item {loop_page === $page && 'active'}">
                <button class="page-link" on:click="{() => get_question_list(loop_page)}">{loop_page+1}</button>
            </li>
        <!-- 첫번째/마지막 페이지가 아닌 경우, 현재 페이지 전후 3개 페이지를 표시 -->
        {:else if loop_page > 0 && loop_page >= $page-3 && loop_page <= $page+3 && loop_page < total_page-1}
            <!-- 현재 페이지가 5번 이상인 경우 1번째 페이지와 현재페이지-3 페이지 사이에 ... 등록 -->
            {#if loop_page === $page-3 && loop_page > 1}
            <li class="page-item {'disabled'}">
                <button class="page-link">...</button>
            </li>
            {/if}
            <li class="page-item {loop_page === $page && 'active'}">
                <button class="page-link" on:click="{() => get_question_list(loop_page)}">{loop_page+1}</button>
            </li>
            <!-- 마찬가지로 마지막 페이지와 현재 페이지+3 페이지 사이에 ... 입력 -->
            {#if loop_page === $page+3 && loop_page < total_page-2}
            <li class="page-item {'disabled'}">
                <button class="page-link">...</button>
            </li>
            {/if}        
        {/if}
        {/each}
        <!-- 다음 페이지 -->
        <li class="page-item {$page >= total_page-1 && 'disabled'}">
            <button class="page-link" on:click="{() => get_question_list($page+1)}">다음</button>
        </li>
    </ul>
    <!-- 페이징 처리 끝 -->
    <a use:link href="/question-create" class="btn btn-primary">질문 등록하기</a>
</div>