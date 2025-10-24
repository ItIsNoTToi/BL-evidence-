<script lang="js">
    import { getQueryFunction } from '@evidence-dev/component-utilities/buildQuery';
    import { Query } from '@evidence-dev/sdk/usql';
    import { BarChart, QueryLoad  } from '@evidence-dev/core-components';

    let queryUserStatus;
    let queryUserDetails;
    let selectedDate = '2024-03-02T17:00:00.000Z';
    let selectedDepartment = '';

    const queryFnUserStatus = Query.createReactive({
        execFn: getQueryFunction(),
        callback: v => (queryUserStatus = v)
    });

    const queryFnUserDetails = Query.createReactive({
        execFn: getQueryFunction(),
        callback: v => (queryUserDetails = v)
    });

    $: queryFnUserStatus(`
        SELECT 
            jsonb_data ->> 'department' AS department,
            DATE_TRUNC('hour', (jsonb_data ->> 'created_date')::timestamp) AS hour,
            COUNT(*) FILTER (WHERE jsonb_data ->> 'status' = 'published') AS active_users,
            COUNT(*) FILTER (WHERE jsonb_data ->> 'status' = 'approved') AS login_users,
            COUNT(*) FILTER (WHERE jsonb_data ->> 'status' = 'review') AS logout_users
        FROM mydatabase.huge_data_table
        WHERE (jsonb_data ->> 'created_date')::timestamp = '${selectedDate}'
        GROUP BY department, hour
        ORDER BY hour;
    `);

     $: if (selectedDepartment) {
        queryFnUserDetails(`
            SELECT
                jsonb_data ->> 'type' AS user_id,
                jsonb_data ->> 'title' AS username,
                jsonb_data ->> 'status' AS status,
                jsonb_data ->> 'department' AS department,
                jsonb_data ->> 'created_date' AS time
            FROM mydatabase.huge_data_table
            WHERE jsonb_data ->> 'department' = '${selectedDepartment}' 
            ORDER BY time DESC;
        `);
    }
</script>


<h2 style="border: 1px solid #fac; padding: 0.5rem 1px; text-align: center; border-radius: 12px; color: white; background-color: #fac;">User Analysis Report</h2>
<QueryLoad data={queryUserStatus} let:loaded={data}>
<svelte:fragment slot="skeleton" />
    <div style="display: grid; grid-template-columns: repeat(3, auto); gap: 20px; margin-top: 20px; border: 1px solid #ccc; padding: 10px; width: fit-content; font-size: 0.8rem; ">
        <div>Total Active Users: {data[0].active_users}</div>
        <div>Total User Login: {data[0].login_users}</div>
        <div>Total User Logout: {data[0].logout_users}</div>
    </div> 
    <div style="margin-top: 20px; display: flex; flex-direction: row; gap: 10px; border: 1px solid #ccc; padding: 10px; width: fit-content; font-size: 0.8rem; ">
        <h2>Chọn phòng ban:</h2>
        <select bind:value={selectedDepartment}>
            <option value="">-- Chọn sở --</option>
            {#each [...new Set(data.map(d => d.department))] as dep}
                <option value={dep}>{dep}</option>
            {/each}
        </select>
        <h2>Chọn ngày:</h2>
        <input bind:value={selectedDate} type="date" />
        <h2>Chọn trạng thái:</h2>
        <select>
            <option value="">-- Chọn trạng thái --</option>
            <option value="published">Published</option>
            <option value="approved">Approved</option>
            <option value="review">Review</option>
        </select>
    </div>

<BarChart
    data={data.filter(d => !selectedDepartment || d.department === selectedDepartment)}
    x={"hour"}
    y={["login_users", "logout_users"]}
    title={`User Login/Logout theo thời gian ${selectedDepartment ? '- ' + selectedDepartment : ''}`}
/>
</QueryLoad>

{#if selectedDepartment}
    <h3 style="margin-top: 20px; font-size: 0.8rem; ">Danh sách user login/logout của {selectedDepartment}</h3>
    <QueryLoad data={queryUserDetails} let:loaded={detailData}>
        <svelte:fragment slot="skeleton">Đang tải dữ liệu...</svelte:fragment>
        <div style="margin-top: 20px; border: 1px solid #ccc; border-radius: 8px; padding: 10px;">
            <table style="width: 100%; border-collapse: collapse;">
                <thead style="background-color: #f2f2f2;">
                    <tr>
                        <th style="border: 1px solid #ddd; padding: 8px;">User ID</th>
                        <th style="border: 1px solid #ddd; padding: 8px;">Username</th>
                        <th style="border: 1px solid #ddd; padding: 8px;">Status</th>
                        <th style="border: 1px solid #ddd; padding: 8px;">Department</th>
                        <th style="border: 1px solid #ddd; padding: 8px;">Time</th>
                    </tr>
                </thead>
                <tbody>
                    {#each detailData as row}
                        <tr>
                            <td style="border: 1px solid #ddd; padding: 8px;">{row.user_id}</td>
                            <td style="border: 1px solid #ddd; padding: 8px;">{row.username}</td>
                            <td style="border: 1px solid #ddd; padding: 8px;">{row.status}</td>
                            <td style="border: 1px solid #ddd; padding: 8px;">{row.department}</td>
                            <td style="border: 1px solid #ddd; padding: 8px;">{row.time}</td>
                        </tr>
                    {/each}
                </tbody>
            </table>
        </div>
    </QueryLoad>
{/if}