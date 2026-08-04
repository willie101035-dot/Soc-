# VGA_pingpong_ball

## 程式碼
### top_moudle
``` vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity top_module is
    Port (
        clk_25MHz   : in  STD_LOGIC;                    -- 接收板載 100MHz 時脈
        rst         : in  STD_LOGIC;                    -- 重置訊號
        btn_p1_up   : in  STD_LOGIC;
        btn_p1_down : in  STD_LOGIC;
        btn_p2_up   : in  STD_LOGIC;
        btn_p2_down : in  STD_LOGIC;
        hsync       : out STD_LOGIC;
        vsync       : out STD_LOGIC;
        vga_rgb     : out STD_LOGIC_VECTOR(11 downto 0)
    );
end top_module;

architecture Structural of top_module is

    component sub_moudle_sync is
        port(
            clk_25MHz        : in  STD_LOGIC;
            rst              : in  STD_LOGIC;
            hsync            : out STD_LOGIC;
            vsync            : out STD_LOGIC;
            disp_en_internal : out STD_LOGIC;
            pixel_x          : out integer range 0 to 799;
            pixel_y          : out integer range 0 to 524
        );
    end component;

    component sub_moudle_game is
        Port (
            clk_25MHz        : in  STD_LOGIC;
            rst              : in  STD_LOGIC;
            disp_en_internal : in  STD_LOGIC;
            pixel_x          : in  INTEGER range 0 to 799;
            pixel_y          : in  INTEGER range 0 to 524;
            btn_p1_up        : in  STD_LOGIC;
            btn_p1_down      : in  STD_LOGIC;
            btn_p2_up        : in  STD_LOGIC;
            btn_p2_down      : in  STD_LOGIC;
            rgb              : out STD_LOGIC_VECTOR(11 downto 0)
        );
    end component;

    -- 內部除頻訊號 (把 100MHz 除頻成真正的 25MHz)
    signal real_clk_25MHz : STD_LOGIC := '0';
    signal clk_divider    : unsigned(1 downto 0) := "00";

    signal disp_en_internal : STD_LOGIC;
    signal pixel_x          : INTEGER range 0 to 799;
    signal pixel_y          : INTEGER range 0 to 524;

begin

    process(clk_25MHz)
    begin
        if rising_edge(clk_25MHz) then
            clk_divider <= clk_divider + 1;
        end if;
    end process;
    
    real_clk_25MHz <= clk_divider(1); -- 第 2 位元就是 25MHz

    u_sync : sub_moudle_sync
        port map (
            clk_25MHz        => real_clk_25MHz,  -- 傳入 25MHz
            rst              => rst,
            hsync            => hsync,
            vsync            => vsync,
            disp_en_internal => disp_en_internal,
            pixel_x          => pixel_x,
            pixel_y          => pixel_y
        );

    u_game : sub_moudle_game
        port map (
            clk_25MHz        => real_clk_25MHz,  -- 傳入 25MHz
            rst              => rst,
            disp_en_internal => disp_en_internal,
            pixel_x          => pixel_x,
            pixel_y          => pixel_y,
            btn_p1_up        => btn_p1_up,
            btn_p1_down      => btn_p1_down,
            btn_p2_up        => btn_p2_up,
            btn_p2_down      => btn_p2_down,
            rgb              => vga_rgb
        );

end Structural;
```
### sub_moudle_sync
```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity sub_moudle_sync is
	port(
		clk_25MHz : in STD_LOGIC;
		rst : in STD_LOGIC;
		hsync : out STD_LOGIC;
		vsync : out STD_LOGIC;
		disp_en_internal : out STD_LOGIC;
		pixel_x : out integer range 0 to 799;
		pixel_y : out integer range 0 to 524
	);
end sub_moudle_sync;

architecture Behavioral of sub_moudle_sync is
	constant HD : integer := 640;
    constant HF : integer := 16;  
    constant HB : integer := 96;  
    constant HR : integer := 48;  
    constant HT : integer := 800; 

    constant VD : integer := 480; 
    constant VF : integer := 10;  
    constant VB : integer := 2;   
    constant VR : integer := 33;  
    constant VT : integer := 525; 
	
	signal h_count : integer range 0 to HT-1 := 0;
    signal v_count : integer range 0 to VT-1 := 0;
	
begin

	process(clk_25MHz,rst)
	begin
		if rst = '1' then
			h_count <= 0;
		elsif rising_edge (clk_25MHz) then
			if h_count = (HT-1) then
				h_count <= 0;
			else
				h_count <= h_count + 1;
			end if;
		end if;
	end process;
	
	process(clk_25MHz,rst)
	begin
		if rst = '1' then
			v_count <= 0;
		elsif rising_edge(clk_25MHz) then
			if h_count = (HT-1) then
				if v_count = (VT-1) then
					v_count <= 0;
				else
					v_count <= v_count + 1;
				end if;
			end if;
		end if;
	end process;
	
	process(clk_25MHz,rst)
	begin
		if rst = '1' then
			hsync <= '0';
		elsif rising_edge(clk_25MHz) then
			if (h_count >= HD + HF) and (h_count < HD + HF +HB ) then
				hsync <= '1' ;
			else
				hsync <= '0';
			end if;
		end if;
	end process;
	
	process(clk_25MHz,rst)
	begin
		if rst = '1' then
			vsync <= '0';
		elsif rising_edge(clk_25MHz) then
			if (v_count >= VD + VF) and (v_count < VD + VF +VB ) then
				vsync <= '1' ;
			else
				vsync <= '0';
			end if;
		end if;
	end process;
	
	process(clk_25MHz,rst)
	begin
		if rst = '1' then
			disp_en_internal <= '0' ;
		elsif rising_edge(clk_25MHz) then
			if (h_count < HD) and (v_count < VD) then
				disp_en_internal <= '1';
			else
				disp_en_internal <= '0' ;
			end if;
		end if;
	end process;
	
	pixel_x <= h_count;
	pixel_y <= v_count;
	
end Behavioral;
```
### sub_moudle_game
```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL; -- 修正：補上句點

entity sub_moudle_game is
    Port (
        clk_25MHz        : in  STD_LOGIC;
        rst              : in  STD_LOGIC;        -- 高準位觸發重置 (Active-High)
        disp_en_internal : in  STD_LOGIC;
        pixel_x          : in  INTEGER range 0 to 799;
        pixel_y          : in  INTEGER range 0 to 524;
        btn_p1_up        : in  STD_LOGIC;
        btn_p1_down      : in  STD_LOGIC;
        btn_p2_up        : in  STD_LOGIC;
        btn_p2_down      : in  STD_LOGIC;
        rgb              : out STD_LOGIC_VECTOR(11 downto 0)
    );
end sub_moudle_game;

architecture Behavioral of sub_moudle_game is
    -- 常量宣告
    constant MAX_X    : integer := 640; -- 補上 MAX_X 宣告
    constant MAX_Y    : integer := 480;
    constant MIN_Y    : integer := 0;

    constant PADDLE_W : integer := 10;
    constant PADDLE_H : integer := 60;
    constant BALL_SIZE: integer := 8;

    constant P1_X     : integer := 20;
    constant P2_X     : integer := 610;

    -- 球拍位置訊號
    signal p1_y : integer range 0 to MAX_Y := 210;
    signal p2_y : integer range 0 to MAX_Y := 210;
    
    -- 球體位置與速度訊號
    signal ball_x      : integer range -100 to 800 := 316;
    signal ball_y      : integer range -100 to 800 := 236;
    signal ball_x_next : integer range -100 to 800; -- 補上訊號宣告
    signal ball_y_next : integer range -100 to 800; -- 補上訊號宣告
    
    signal ball_dx     : integer := 2;
    signal ball_dy     : integer := 2;

    signal frame_tick  : std_logic;

begin

    ------------------------------------------------------------------
    -- 1. Frame Tick 產生器 (每幀 60Hz 觸發一次更新)
    ------------------------------------------------------------------
    process(clk_25MHz, rst)
    begin
        if rst = '1' then
            frame_tick <= '0';
        elsif rising_edge(clk_25MHz) then
            if (pixel_x = 0) and (pixel_y = 0) then
                frame_tick <= '1';
            else
                frame_tick <= '0';
            end if;
        end if;
    end process;

    ------------------------------------------------------------------
    -- 2. P1 與 P2 球拍移動邏輯
    ------------------------------------------------------------------
    -- P1 球拍
    process(clk_25MHz, rst)
    begin
        if rst = '1' then
            p1_y <= 210;
        elsif rising_edge(clk_25MHz) then
            if frame_tick = '1' then
                if (btn_p1_up = '1') and (p1_y >= 10) then
                    p1_y <= p1_y - 4;
                elsif (btn_p1_down = '1') and (p1_y + PADDLE_H <= MAX_Y - 10) then
                    p1_y <= p1_y + 4;
                end if;
            end if;
        end if;
    end process;

    -- P2 球拍
    process(clk_25MHz, rst)
    begin
        if rst = '1' then
            p2_y <= 210;
        elsif rising_edge(clk_25MHz) then
            if frame_tick = '1' then
                if (btn_p2_up = '1') and (p2_y >= 10) then
                    p2_y <= p2_y - 4;
                elsif (btn_p2_down = '1') and (p2_y + PADDLE_H <= MAX_Y - 10) then
                    p2_y <= p2_y + 4;
                end if;
            end if;
        end if;
    end process;

    ------------------------------------------------------------------
    -- 3. 組合邏輯：預算下一幀的球體座標 (ball_x_next / ball_y_next)
    ------------------------------------------------------------------
    process(rst, ball_y, ball_dy)
    begin 
        if rst = '1' then
            ball_y_next <= 236;
        else 
            ball_y_next <= ball_y + ball_dy;
        end if;
    end process;

    process(rst, ball_x, ball_dx)
    begin 
        if rst = '1' then
            ball_x_next <= 316;
        else
            ball_x_next <= ball_x + ball_dx;
        end if;
    end process;

    ------------------------------------------------------------------
    -- 4. 時序邏輯：更新球體 X, Y 座標與出界重置
    ------------------------------------------------------------------
    -- Y 軸位置更新
    process(clk_25MHz, rst)
    begin 
        if rst = '1' then
            ball_y <= 236;
        elsif rising_edge(clk_25MHz) then 
            if frame_tick = '1' then
                if (ball_x_next + BALL_SIZE <= MAX_X) and (ball_x_next >= 0) then
                    ball_y <= ball_y_next;
                else
                    ball_y <= 236; -- 出界時回歸中央
                end if;
            end if;
        end if;
    end process;

    -- X 軸位置更新
    process(clk_25MHz, rst)
    begin
        if rst = '1' then
            ball_x <= 316;
        elsif rising_edge(clk_25MHz) then 
            if frame_tick = '1' then
                if (ball_x_next + BALL_SIZE <= MAX_X) and (ball_x_next >= 0) then
                    ball_x <= ball_x_next;
                else
                    ball_x <= 316; -- 出界時回歸中央
                end if;
            end if;
        end if;
    end process;

    ------------------------------------------------------------------
    -- 5. 球體碰撞反彈邏輯 (Y 軸牆壁 & X 軸球拍)
    ------------------------------------------------------------------
    -- Y 軸上下牆壁反彈
    process(clk_25MHz, rst)
    begin 
        if rst = '1' then
            ball_dy <= 2;
        elsif rising_edge(clk_25MHz) then 
            if frame_tick = '1' then
                if (ball_y_next <= MIN_Y) and (ball_dy < 0) then
                    ball_dy <= 2;
                elsif (ball_y_next + BALL_SIZE >= MAX_Y) and (ball_dy > 0) then 
                    ball_dy <= -2;
                end if;
            end if;
        end if;
    end process;

    -- X 軸球拍碰撞反彈
    process(clk_25MHz, rst)
    begin
        if rst = '1' then 
            ball_dx <= 2;
        elsif rising_edge(clk_25MHz) then
            if frame_tick = '1' then
                -- P2 右邊球拍碰撞判定
                if (ball_x_next + BALL_SIZE >= P2_X) and (ball_x_next <= P2_X + PADDLE_W) and 
                   (ball_y_next + BALL_SIZE >= p2_y) and (ball_y_next <= p2_y + PADDLE_H) then
                    ball_dx <= -2;
                -- P1 左邊球拍碰撞判定
                elsif (ball_x_next <= P1_X + PADDLE_W) and (ball_x_next + BALL_SIZE >= P1_X) and 
                      (ball_y_next + BALL_SIZE >= p1_y) and (ball_y_next <= p1_y + PADDLE_H) then
                    ball_dx <= 2;
                end if;
            end if;
        end if;
    end process;

    ------------------------------------------------------------------
    -- 6. VGA 畫面渲染 (RGB 繪圖模組)
    ------------------------------------------------------------------
    process(disp_en_internal, pixel_x, pixel_y, p1_y, p2_y, ball_x, ball_y)
        variable is_p1   : boolean;
        variable is_p2   : boolean;
        variable is_ball : boolean;
    begin
        if disp_en_internal = '0' then
            rgb <= (others => '0');
        else
            -- 物件範圍判定
            is_p1 := (pixel_x >= P1_X) and (pixel_x < P1_X + PADDLE_W) and 
                     (pixel_y >= p1_y) and (pixel_y < p1_y + PADDLE_H);

            is_p2 := (pixel_x >= P2_X) and (pixel_x < P2_X + PADDLE_W) and 
                     (pixel_y >= p2_y) and (pixel_y < p2_y + PADDLE_H);

            is_ball := (pixel_x >= ball_x) and (pixel_x < ball_x + BALL_SIZE) and 
                       (pixel_y >= ball_y) and (pixel_y < ball_y + BALL_SIZE);

            -- RGB 色彩輸出
            if is_p1 then
                rgb <= X"F00"; -- P1：紅色
            elsif is_p2 then
                rgb <= X"00F"; -- P2：藍色
            elsif is_ball then
                rgb <= X"FFF"; -- 球：白色
            else
                rgb <= X"000"; -- 背景：黑色
            end if;
        end if;
    end process;

end Behavioral;
```
## FPGA板
https://github.com/user-attachments/assets/44594ca0-54e6-4b2e-9cc2-3732b0666f4b

