// Các cấu hình manifest (thường nằm ở file plugin.json hoặc manifest.json đi kèm, 
// nhưng trong JS, chúng ta dùng các hàm chuẩn của SDK)

const BASE_URL = "https://phim.nguonc.com";

/**
 * 1. TRANG CHỦ (getHome)
 * SkyStream gọi hàm này đầu tiên. Dùng callback (cb) để đẩy từng hàng phim ra màn hình.
 */
async function getHome(cb) {
    const sections = [
        { title: "🎬 Phim Mới Cập Nhật", path: "/api/films/phim-moi-cap-nhat?page=1" },
        { title: "🍿 Phim Lẻ Mới", path: "/api/films/danh-sach/phim-le?page=1" },
        { title: "📺 Phim Bộ Đang Chiếu", path: "/api/films/danh-sach/phim-bo?page=1" }
    ];

    try {
        for (const sec of sections) {
            const res = await fetch(BASE_URL + sec.path);
            const data = await res.json();
            
            let items = [];
            if (data && data.items) {
                items = data.items.map(film => ({
                    title: film.name,
                    url: film.slug, 
                    posterUrl: film.thumb_url
                }));
            }
            
            // SkyStream yêu cầu dùng cb để đẩy từng section ra giao diện
            // Đẩy đến đâu màn hình sẽ load đến đó (Lazy load)
            cb({
                title: sec.title,
                items: items
            });
        }
    } catch (error) {
        console.error("Lỗi getHome NguonC: " + error);
    }
}

/**
 * 2. TÌM KIẾM (search)
 * Trả kết quả tìm kiếm thông qua cb
 */
async function search(query, cb) {
    try {
        const url = `${BASE_URL}/api/films/search?keyword=${encodeURIComponent(query)}`;
        const res = await fetch(url);
        const data = await res.json();
        
        let items = [];
        if (data && data.items) {
            items = data.items.map(film => ({
                title: film.name,
                url: film.slug,
                posterUrl: film.thumb_url
            }));
        }
        
        // Trả kết quả về
        cb(items);
    } catch (error) {
        console.error("Lỗi search NguonC: " + error);
        cb([]);
    }
}

/**
 * 3. CHI TIẾT PHIM (loadDetail)
 * Lấy thông tin phim và danh sách tập
 */
async function loadDetail(url, cb) {
    try {
        // url truyền vào chính là slug của phim
        const apiUrl = `${BASE_URL}/api/film/${url}`;
        const res = await fetch(apiUrl);
        const data = await res.json();
        
        if (!data || !data.film) {
            cb(null);
            return;
        }
        
        const film = data.film;
        let episodes = [];
        
        if (film.episodes) {
            film.episodes.forEach(server => {
                const serverName = server.server_name || "Server";
                server.items.forEach(ep => {
                    episodes.push({
                        name: `[${serverName}] - Tập ${ep.name}`,
                        // Gom chung link m3u8 vào url để truyền sang bước stream
                        url: ep.link_m3u8 || ep.link_embed 
                    });
                });
            });
        }
        
        // Đẩy toàn bộ cục dữ liệu chi tiết cho SkyStream dựng UI
        cb({
            title: film.name,
            posterUrl: film.thumb_url,
            // Xóa sạch các thẻ HTML rác khỏi đoạn mô tả
            description: film.content ? film.content.replace(/<[^>]*>?/gm, '') : "",
            status: film.episode_current,
            episodes: episodes
        });
        
    } catch (error) {
        console.error("Lỗi loadDetail NguonC: " + error);
        cb(null);
    }
}

/**
 * 4. TRÍCH XUẤT LUỒNG PHÁT (loadStreams)
 * Dùng cb để ném link video cho Player phát.
 */
async function loadStreams(url, cb) {
    try {
        // url ở đây chính là link_m3u8 đã lấy từ hàm loadDetail
        // Vì NguonC trả link HLS chuẩn, không cần dùng skystream-extractors để bóc tách
        const stream = {
            url: url,
            title: "NguonC Stream",
            isM3U8: url.includes(".m3u8")
        };
        
        cb(stream);
    } catch (error) {
        console.error("Lỗi loadStreams NguonC: " + error);
        cb(null);
    }
}

// Bắt buộc phải có trong SkyStream để ứng dụng có thể gọi các hàm này
module.exports = {
    getHome,
    search,
    loadDetail,
    loadStreams
};
