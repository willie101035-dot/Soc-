# VGA
```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity vga_sync is
    Port (
        clk_25MHz : in  STD_LOGIC;                    
        rst       : in  STD_LOGIC;                     
        h_sync    : out STD_LOGIC;                     
        v_sync    : out STD_LOGIC;                     
        disp_en   : out STD_LOGIC;
        vga_r     : out STD_LOGIC_VECTOR(2 downto 0); 
        vga_g     : out STD_LOGIC_VECTOR(2 downto 0); 
        vga_b     : out STD_LOGIC_VECTOR(2 downto 0);     
        pixel_x   : out STD_LOGIC_VECTOR(9 downto 0);
        pixel_y   : out STD_LOGIC_VECTOR(9 downto 0)    
    );
end vga_sync;

architecture Behavioral of vga_sync is
    constant HD : integer := 640; 
    constant HF : integer := 16;  
    constant HS : integer := 96;  
    constant HB : integer := 48; 
    constant HT : integer := 800; 

    constant VD : integer := 480; 
    constant VF : integer := 10;  
    constant VS : integer := 2;   
    constant VB : integer := 33;  
    constant VT : integer := 525;

    signal h_cnt_reg, h_cnt_next : unsigned(9 downto 0) := (others => '0');
    signal v_cnt_reg, v_cnt_next : unsigned(9 downto 0) := (others => '0');

    signal h_sync_reg, v_sync_reg : STD_LOGIC := '1';
    signal disp_en_internal       : STD_LOGIC;
begin

    process(clk_25MHz, rst)
    begin
        if rst = '1' then
            h_cnt_reg <= (others => '0');
        elsif rising_edge(clk_25MHz) then
            h_cnt_reg <= h_cnt_next;
        end if;
    end process;

    process(clk_25MHz, rst)
    begin
        if rst = '1' then
            v_cnt_reg <= (others => '0');
        elsif rising_edge(clk_25MHz) then
            v_cnt_reg <= v_cnt_next;
        end if;
    end process;

    process(clk_25MHz, rst)
    begin
        if rst = '1' then
            h_sync_reg <= '1';
        elsif rising_edge(clk_25MHz) then
            if (h_cnt_reg >= (HD + HF) and h_cnt_reg < (HD + HF + HS)) then
                h_sync_reg <= '0';
            else
                h_sync_reg <= '1';
            end if; 
        end if;
    end process;

    process(clk_25MHz, rst)
    begin
        if rst = '1' then
            v_sync_reg <= '1';
        elsif rising_edge(clk_25MHz) then
            if (v_cnt_reg >= (VD + VF) and v_cnt_reg < (VD + VF + VS)) then
                v_sync_reg <= '0';
            else
                v_sync_reg <= '1';
            end if; 
        end if;
    end process;

    disp_en_internal <= '1' when (h_cnt_reg < HD and v_cnt_reg < VD) else '0';

    process(h_cnt_reg)
    begin
        if h_cnt_reg = (HT - 1) then
            h_cnt_next <= (others => '0');
        else 
            h_cnt_next <= h_cnt_reg + 1;
        end if;
    end process;

    process(v_cnt_reg, h_cnt_reg)
    begin
        v_cnt_next <= v_cnt_reg; 
        if h_cnt_reg = (HT - 1) then
            if v_cnt_reg = (VT - 1) then
                v_cnt_next <= (others => '0');
            else
                v_cnt_next <= v_cnt_reg + 1;
            end if;
        end if;
    end process;

    process(disp_en_internal)
    begin
        if disp_en_internal = '1' then
            vga_r <= "111";
            vga_g <= "000";
            vga_b <= "111";
        else
            vga_r <= "000";
            vga_g <= "000";
            vga_b <= "000";
        end if;
    end process;

    h_sync  <= h_sync_reg;
    v_sync  <= v_sync_reg;
    disp_en <= disp_en_internal;

    pixel_x <= std_logic_vector(h_cnt_reg);
    pixel_y <= std_logic_vector(v_cnt_reg);

end Behavioral;
```
# 上板子
<img width="1477" height="1108" alt="S__38526983" src="https://github.com/user-attachments/assets/8ad281c2-1fdb-4fae-a639-d112b8ba0cca" />
