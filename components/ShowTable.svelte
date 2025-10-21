<script lang="js">
    import { DownloadData, QueryLoad } from '@evidence-dev/core-components';
    import { getQueryFunction } from '@evidence-dev/component-utilities/buildQuery';
    import { Query } from '@evidence-dev/sdk/usql';

    let page = 1;
    let limit = 15;
    let queryData, queryTotal;

    let showDetail = false;
    let selectedData = null;

    const queryFnData = Query.createReactive({
        execFn: getQueryFunction(),
        callback: v => (queryData = v)
    });

    const queryFnTotal = Query.createReactive({
        execFn: getQueryFunction(),
        callback: v => (queryTotal = v)
    });

    $: queryFnData(`
        SELECT *
        FROM mydatabase.orders
        ORDER BY created_at DESC
        LIMIT ${limit}
        OFFSET ${(page - 1) * limit};
    `);

    $: queryFnTotal(`
        SELECT COUNT(*) AS total_orders
        FROM mydatabase.orders;
    `);

    const parseJson = (item) => {
        if (!item) return null;
        try {
            return JSON.parse(JSON.parse(item));
        } catch (e) {
            console.error("Error parsing JSONB data:", e);
            return null;
        }
    };

    function goToPage(totalPages) {
        if (page < totalPages) page++;
    }
    function backToPage() {
        if (page > 1) page--;
    }

    function reloadPage() {
        window.location.reload();
    }

    function viewDetails(item) {
        selectedData = parseJson(item);
        showDetail = true;
    }

    function closeDetail() {
        showDetail = false;
        selectedData = null;
    }

    function formatDate(dateStr) {
        if (!dateStr) return 'N/A';
        const date = new Date(dateStr);
        if (isNaN(date)) return dateStr; // nếu không parse được thì trả về nguyên chuỗi
        return date.toLocaleDateString('vi-VN', {
            day: '2-digit',
            month: '2-digit',
            year: 'numeric'
        });
    }
</script>

<QueryLoad data={queryData} let:loaded={data}>
    <svelte:fragment slot="skeleton" />
    <div class="table-container">
        <table class="data-table">
            <thead>
                <tr>
                    <th>STT</th>
                    <th>Mã định danh</th>
                    <th>Nơi tạo bản tin</th>
                    <th>Số thông báo</th>
                    <th>Ngày cấp</th>
                    <th>Nguồn dữ liệu</th>
                    <th>Trạng thái</th>
                    <th>Thao tác</th>
                </tr>
            </thead>
            <tbody>
                {#if data.length > 0}
                    {#each data as item, index}
                        <tr>
                            <td>{(page - 1) * limit + index + 1}</td>
                            <td>{item.madinhdanh}</td>
                            <td>{parseJson(item.jsonb_data)?.DacTaBanTin?.NoiTaoBanTin || 'N/A'}</td>
                            <td>{parseJson(item.jsonb_data)?.DuLieuBanTin[0]?.SoThongBao || 'N/A'}</td>
                            <td>{formatDate(parseJson(item.jsonb_data)?.DuLieuBanTin[0]?.NgayCap) || 'N/A'}</td>
                            <td>{item.source_ref}</td>
                            <td>{item.is_valid ? 'Hợp lệ' : 'Không hợp lệ'}</td>
                            <td class="actions">
                                <button on:click={() => viewDetails(item.jsonb_data)}>🔍</button>
                                <DownloadData data={data} text=''/>
                                <button on:click={reloadPage}>🔄️</button>
                            </td>
                        </tr>
                    {/each}
                {:else}
                    <tr><td colspan="8" style="text-align:center;">Không có dữ liệu</td></tr>
                {/if}
            </tbody>
        </table>

        <QueryLoad data={queryTotal} let:loaded={total}>
            {@const totalPages = Math.ceil(total[0].total_orders / limit)}
            <div class="pagination-bar">
                <div class="page-controls">
                    <button on:click={backToPage} disabled={page <= 1}>{'<'}</button>
                    <div>Trang {page} / {totalPages}</div>
                    <button on:click={goToPage(totalPages)} disabled={page >= totalPages}>{'>'}</button>
                </div>
                <div>Tổng bản tin: {total[0].total_orders}</div>
                <div>Hiển thị:
                    <select bind:value={limit} on:change={() => { page = 1; }}>
                        <option value={15}>15</option>
                        <option value={30}>30</option>
                        <option value={50}>50</option>
                    </select>
                </div>
            </div>
        </QueryLoad>
    </div>

    {#if showDetail}
        <div class="overlay" on:click={closeDetail}>
            <div class="detail-popup" on:click|stopPropagation>
                <button class="close-btn" on:click={closeDetail}>✖</button>
                <h2>Chi tiết bản tin</h2>
                <pre>{JSON.stringify(selectedData, null, 2)}</pre>
            </div>
        </div>
    {/if}
</QueryLoad>

<style>
.table-container {
    width: 100%;
    overflow-x: auto;
    margin-top: 1rem;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}
.data-table {
    width: 100%;
    border-collapse: collapse;
}
.data-table thead {
    background-color: #2363f0;
    color: white;
}
.data-table th, .data-table td {
    border: 1px solid #e5e7eb;
    padding: 0.5rem;
    font-size: 0.5rem;
    text-align: center;
}
.actions {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 0.5rem;
}
.pagination-bar {
    display: flex;
    justify-content: space-around;
    margin: 1rem 0px;
    font-size: 0.5rem;
    align-items: center;
}
.page-controls {
    display: flex;
    align-items: center;
}
.page-controls button {
    padding: 0.3rem 0.6rem;
    background-color: #2563eb;
    color: white;
    border: none;
    cursor: pointer;
}
.page-controls div {
    margin: 0 0.5rem;
}
.overlay {
    position: fixed;
    top: 0; left: 0; right: 0; bottom: 0;
    background: rgba(0,0,0,0.5);
    z-index: 1000;
    display: flex;
    justify-content: center;
    align-items: center;
}
.detail-popup {
    background: #fff;
    padding: 1.5rem;
    border-radius: 0.5rem;
    width: 70%;
    max-height: 80%;
    overflow-y: auto;
    position: relative;
}
.close-btn {
    position: absolute;
    top: 10px; right: 10px;
    background: #ef4444;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    padding: 0.2rem 0.4rem;
}
pre {
    background: #f9fafb;
    padding: 1rem;
    border-radius: 0.5rem;
    overflow-x: auto;
    font-size: 0.8rem;
}
</style>
